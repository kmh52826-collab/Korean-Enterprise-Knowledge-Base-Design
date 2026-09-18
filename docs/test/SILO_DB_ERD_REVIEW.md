# 회사별 Silo DB 전제의 ERD 정합성 검토 및 개선 제안

- 검토 기준: 고객사별 Silo DB, **73개 테이블·1,177개 컬럼**, 14개 ERD
- 기준 자료: [데이터 테이블 정의서](../data-dictionary.md), [ERD 명세서](../erd-specifications.md), [전체 데이터 구조도](../erd-overview.md)
- 검토 범위: 데이터 구조와 관계의 정합성. 실제 운영 환경의 고객사 격리 시험은 별도 수행
- 예시 기준: A제약·B제약은 가상 고객사이며 아래 데이터는 설명용이다. ID는 읽기 쉽게 줄였으며 실제 명세의 자료형은 UUID다. 예시 표에는 설명에 필요한 컬럼만 표시했다.

## 1. 검토 결론

**회사별 DB를 분리하더라도, 회사 안의 본사·공장·연구소를 구분할 필요가 있다면 `organization_id`는 유지하는 것이 적절하다.**

현재 명세에서는 사용자·사용자 그룹·시스템이 조직을 참조한다. 이 연결은 소속을 표시하고 조직별 데이터를 조회하는 데 사용한다. 회사별 Silo와 충돌해 반드시 삭제해야 하는 테이블·컬럼·FK는 확인되지 않았다.

| 구분 | 이 문서의 예시 | 담당하는 구조 |
|---|---|---|
| 고객사 분리 | A제약과 B제약의 데이터 분리 | 고객사별 전용 DB 환경과 인증·접속 제어 |
| 회사 내부 조직 구분 | A제약 본사와 A제약 생산공장 구분 | 각 회사 DB 안의 `organization` |
| 업무 참여와 권한 | 본사 담당자가 공장 프로젝트를 검토 | 프로젝트 참여·접근권한·결재 담당 배정 |

회사별 Silo는 정해진 전제이고, **`organization`을 고객사 내부 조직으로 명확히 정의하는 것은 이 검토의 권고안**이다. 현재 명세에는 고객사와 내부 조직이라는 표현이 혼용되어 있어 설명 보완이 필요하다.

## 2. 회사별 DB에는 어떤 데이터가 들어가는가

같은 73개 테이블 구조를 A제약과 B제약의 전용 DB에 각각 구성한다. 한 회사의 테이블 73개를 서로 다른 DB로 나누는 의미가 아니다.

```text
A제약 전용 DB 환경
  └─ A제약 DB
       ├─ organization: A제약 본사, A제약 생산공장
       ├─ app_user: A제약 사용자
       ├─ system_asset: A제약 시스템·장비
       ├─ validation_project: A제약 검증 프로젝트
       └─ 평가·시험·문서·승인·감사·파일 검사 등 나머지 테이블

B제약 전용 DB 환경
  └─ B제약 DB
       ├─ organization: B제약 본사, B제약 연구소
       ├─ app_user: B제약 사용자
       ├─ system_asset: B제약 시스템·장비
       ├─ validation_project: B제약 검증 프로젝트
       └─ A제약과 같은 구조의 나머지 테이블
```

예를 들어 각 DB의 `organization`에는 다음 데이터가 들어간다.

| 저장되는 DB | organization_id | organization_name | organization_type |
|---|---|---|---|
| A제약 DB | ORG-01 | A제약 본사 | 본사 |
| A제약 DB | ORG-02 | A제약 생산공장 | 공장 |
| B제약 DB | ORG-01 | B제약 본사 | 본사 |
| B제약 DB | ORG-02 | B제약 연구소 | 연구소 |

표의 **‘저장되는 DB’는 설명용 구분이며 실제 컬럼이 아니다.** 서로 다른 DB에서는 식별자 값이 같아도 각 DB의 별도 행이다. 같은 값을 사용해야 한다는 뜻은 아니며, 실제 UUID는 각 환경에서 발급한다.

서버는 인증된 고객사에 맞는 DB에 먼저 연결한다. 그다음 해당 DB 안에서 `organization_id`를 조회한다. **A제약 DB에서 ORG-02를 선택하면 A제약 생산공장을 뜻하며 B제약 DB로 전환되지 않는다.**

## 3. A제약 DB 안에서 organization_id를 사용하는 예시

### 3.1 사용자와 그룹의 소속을 확인한다

**조직 목록 — [organization](../data-dictionary.md#table-organization)**

| organization_id | organization_name | organization_type |
|---|---|---|
| ORG-01 | A제약 본사 | 본사 |
| ORG-02 | A제약 생산공장 | 공장 |

**사용자 — [app_user](../data-dictionary.md#table-app_user)**

| user_id | full_name | organization_id |
|---|---|---|
| USER-01 | 김민호 | ORG-01 |
| USER-02 | 홍길동 | ORG-02 |

**사용자 그룹 — [user_group](../data-dictionary.md#table-user_group)**

| user_group_id | group_code | group_name | organization_id |
|---|---|---|---|
| GROUP-01 | QA_REVIEWER | 본사 QA 검토자 | ORG-01 |
| GROUP-02 | QA_REVIEWER | 공장 QA 검토자 | ORG-02 |

JOIN은 **같은 ID로 연결된 두 테이블에서 필요한 정보를 함께 읽는 것**이다. 사용자와 조직을 연결하면 다음 결과를 얻는다.

```sql
SELECT
    u.full_name AS 사용자명,
    o.organization_name AS 소속조직
FROM app_user AS u
JOIN organization AS o
  ON o.organization_id = u.organization_id;
```

| 사용자명 | 소속조직 |
|---|---|
| 김민호 | A제약 본사 |
| 홍길동 | A제약 생산공장 |

그룹도 `user_group.organization_id = organization.organization_id`로 연결하여 다음처럼 조회할 수 있다.

| 그룹명 | 소속조직 |
|---|---|
| 본사 QA 검토자 | A제약 본사 |
| 공장 QA 검토자 | A제약 생산공장 |

그룹의 소속과 실제 그룹 구성원은 별개다. 누가 그룹에 참여하는지는 [user_group_member](../data-dictionary.md#table-user_group_member)에서 관리한다.

### 3.2 프로젝트가 검증하는 시스템의 관리 조직을 확인한다

**시스템·장비 — [system_asset](../data-dictionary.md#table-system_asset)**

| system_id | system_name | management_number | organization_id |
|---|---|---|---|
| SYS-01 | 문서관리 시스템 | SYS-001 | ORG-01 |
| SYS-02 | 제조관리 시스템 | SYS-001 | ORG-02 |

**검증 프로젝트 — [validation_project](../data-dictionary.md#table-validation_project)**

| project_id | project_name | system_id |
|---|---|---|
| PRJ-01 | 제조관리 시스템 밸리데이션 | SYS-02 |

프로젝트에는 `organization_id`가 직접 없다. 프로젝트의 `system_id`로 시스템을 찾고, 시스템의 `organization_id`로 관리 조직을 찾는다.

```text
PRJ-01: 제조관리 시스템 밸리데이션
    │ system_id = SYS-02
    ▼
SYS-02: 제조관리 시스템
    │ organization_id = ORG-02
    ▼
ORG-02: A제약 생산공장
```

```sql
SELECT
    p.project_name AS 프로젝트,
    s.system_name AS 검증대상,
    o.organization_name AS 시스템관리조직
FROM validation_project AS p
JOIN system_asset AS s
  ON s.system_id = p.system_id
JOIN organization AS o
  ON o.organization_id = s.organization_id;
```

| 프로젝트 | 검증대상 | 시스템관리조직 |
|---|---|---|
| 제조관리 시스템 밸리데이션 | 제조관리 시스템 | A제약 생산공장 |

이 조회 결과는 **현재 시스템의 관리 조직**이다. 과거 프로젝트가 채택한 승인 당시의 시스템 정보는 [project_system_baseline](../data-dictionary.md#table-project_system_baseline)과 [system_asset_revision](../data-dictionary.md#table-system_asset_revision)에 보존한 기준으로 확인한다.

위 SQL은 관계를 설명하는 조회 예시다. 실제 서비스에서는 인증된 고객사 DB 연결과 사용자별 조회 권한 검증을 함께 적용해야 한다.

### 3.3 본사 담당자가 공장 프로젝트에 참여할 수 있다

**업무 역할 — [role](../data-dictionary.md#table-role)**

| role_id | role_code | role_name |
|---|---|---|
| ROLE-01 | REVIEWER | 검토자 |

**프로젝트 참여자 — [project_member](../data-dictionary.md#table-project_member)**

| project_member_id | project_id | user_id | role_id | member_status |
|---|---|---|---|---|
| MEMBER-01 | PRJ-01 | USER-01 | ROLE-01 | ACTIVE |

이 데이터를 연결하면 다음 의미가 된다.

| 프로젝트 | 시스템 관리 조직 | 참여자 | 참여자 소속 | 프로젝트 역할 |
|---|---|---|---|---|
| 제조관리 시스템 밸리데이션 | A제약 생산공장 | 김민호 | A제약 본사 | 검토자 |

본사 소속과 공장 시스템의 관리 조직이 달라도 같은 회사 안에서 협업할 수 있다. 따라서 사용자와 프로젝트 대상 시스템의 `organization_id`가 항상 같아야 한다는 규칙을 일괄 적용하면 이런 협업을 막게 된다.

프로젝트에 검토자로 등록한 것만으로 모든 검토 권한이 완성되지는 않는다. 화면·프로젝트 접근은 [access_permission_grant](../data-dictionary.md#table-access_permission_grant), 실제 결재 처리는 [workflow_step_assignee](../data-dictionary.md#table-workflow_step_assignee)의 담당·대체 배정 등 기존 규칙을 함께 확인한다.

### 3.4 조직별 번호 중복을 관리한다

앞의 시스템 예시에서 본사와 공장이 모두 `SYS-001`을 사용할 수 있는 이유는 현재 유일성 기준이 **조직 ID + 관리번호**이기 때문이다.

| 등록 상황 | 현재 명세의 판정 |
|---|---|
| 본사 ORG-01에 SYS-001 등록 | 허용 |
| 공장 ORG-02에 SYS-001 등록 | 허용. 다른 조직의 번호 |
| 본사 ORG-01에 SYS-001을 다시 등록 | 차단. 같은 조직·관리번호 중복 |
| 다른 고객사인 B제약 DB에 SYS-001 등록 | A제약과 독립적으로 B제약 DB의 제약 적용 |

활성 그룹의 `group_code`도 조직별로 중복을 판단한다. 반면 `organization_code`, 사용자 이메일, 프로젝트 코드는 현재 각각 고객사 DB 전체에서 유일하게 관리한다.

회사 전체에서 시스템 관리번호를 중복 없이 사용해야 한다면 그 요구에 맞춰 유일성 제약을 별도 검토한다. **Silo를 사용한다는 이유만으로 기존 제약을 변경하지는 않는다.**

## 4. 현재 ERD에서 유지할 부분과 보완할 부분

### 4.1 직접 연결은 세 곳이다

| FK 컬럼 | 연결 대상 | 실제 사용하는 정보 |
|---|---|---|
| `app_user.organization_id` | `organization.organization_id` | 사용자 소속 |
| `user_group.organization_id` | `organization.organization_id` | 그룹 소속 |
| `system_asset.organization_id` | `organization.organization_id` | 시스템·장비의 관리 조직 |

이 연결을 유지하면 앞의 소속 조회와 조직별 번호 관리가 가능하다. **JOIN이 가능하다는 이유만으로 테이블이 필수인 것은 아니다.** 회사 내부 조직을 구분하는 실제 업무가 있는지가 기준이며, 현재 명세에는 본사·공장·연구소·해외법인 유형과 위 연결이 정의되어 있다.

### 4.2 우선 보완할 것은 회사와 조직의 설명이다

| 현재 명세의 표현 | Silo 전제의 권고 정의 | 구조 변경 |
|---|---|---|
| `organization`: 고객사 또는 운영 조직 | 현재 고객사 DB 안의 운영 조직·사업장 | 설명 보완 |
| `organization_code`: 조직/회사 식별 코드 | 고객사 내부 조직 코드 | 설명 보완 |
| `organization_name`: 회사/사업장명 | 현재 DB에 속한 운영 조직·사업장명 | 설명 보완 |
| `organization_id` | 사용자·그룹·시스템의 내부 조직 연결 키 | 기존 FK 유지 |
| `GLOBAL_ADMIN`·역할 범위 `GLOBAL` | 해당 고객사 안의 전체 관리·역할 범위 | 기존 코드 유지, 범위 명시 |
| 공용 라이브러리·규정·활동 조건 | 해당 고객사 DB 안에서 함께 사용하는 기준정보 | 기존 로컬 참조 유지 |
| 유형+ID 및 JSON 안의 업무 참조 | 같은 고객사 DB 안의 실제 행·개정으로 해석 | 서버 검증 범위 명시 |

‘해외법인’도 계약상 같은 고객사의 격리 범위에 포함될 때 내부 조직으로 등록한다. 별도 고객사로 취급해야 한다면 해당 고객사 전용 DB에 배치한다.

### 4.3 추가 요구가 생기면 구조를 다시 검토한다

| 업무 요구 예시 | 현재 구조로 판단할 수 있는 범위 | 추가 검토 |
|---|---|---|
| “본사·공장·연구소 소속을 표시하고 싶다” | 현재 조직 목록과 FK로 표현 가능 | 기존 구조 유지 |
| “공장 직원은 자기 공장 인벤토리만 볼 수 있어야 한다” | 조직 소속은 알 수 있지만 소속만으로 접근 제한이 생기지는 않음 | 조직 범위의 접근권한 설계 |
| “본사 → 공장 → 부서 계층과 권한 상속이 필요하다” | 현재 `organization`에는 부모 조직 컬럼이 없음 | 조직 계층·권한 상속 설계 |
| “모든 고객사가 하나의 중앙 계정을 사용한다” | 현재 사용자와 역할은 각 업무 DB 안에서 연결됨 | 중앙 인증과 고객사 내부 계정의 연결 |
| “모든 고객사가 중앙 라이브러리 DB를 직접 참조한다” | 현재 FK는 고객사 DB 안의 라이브러리를 참조함 | 중앙 참조·개정 보존 방식 |

현재 `access_permission_grant`는 MENU/PROJECT 범위이고 `inventory_role_grant`에는 대상 조직 컬럼이 없다. 따라서 사업장별 인벤토리 접근 제한까지 현재 구조로 해결됐다고 판단하지 않는다.

## 5. 파일·검사·AI·공통 자료에도 같은 원칙을 적용한다

### 5.1 파일 검사 결과는 업로드한 회사의 DB에 저장한다

검사 서비스를 여러 회사가 함께 사용하더라도 결과는 해당 회사의 DB에서 관리한다.

| 저장되는 DB | scan_job_id | job_status | scan_result | result_file_id |
|---|---|---|---|---|
| A제약 DB | SCAN-A-01 | COMPLETED | NO_THREATS_FOUND | FILE-A-01 |
| B제약 DB | SCAN-B-01 | COMPLETED | THREATS_FOUND | NULL |

A제약 파일은 검사 통과와 원본 동일성 확인을 거쳐 A제약 DB의 `file_asset`에 등록된다. B제약 파일은 검사 작업이 완료됐어도 위협이 발견되어 등록하지 않는다. 이 표는 전체 검사 컬럼 중 판정과 등록 여부만 보여준다.

```text
A제약 업로드 → A제약 검사 작업 → 검사 통과 → A제약 파일 등록
B제약 업로드 → B제약 검사 작업 → 위협 발견 → 사용 차단 유지
```

Worker나 검사 결과 수신 처리는 **서버가 검증한 고객사 정보 + 검사 작업 ID + 검사 원본 객체**를 확인한다. 요청에 적힌 조직 ID나 파일 경로만으로 고객사 DB를 선택하지 않는다.

- 검사 전 파일은 첨부·다운로드·미리보기·문서 파싱·AI 입력에 사용하지 않는다.
- 재시도는 같은 업로드의 새 회차로 기록하고, 중복·지연 응답으로 파일을 두 번 등록하거나 과거 판정을 바꾸지 않는다.
- 파일 자산과 검사 작업의 등록 파일 연결은 같은 DB 트랜잭션으로 확정한다.
- 사용자 업로드는 `file_origin=UPLOAD`, 신뢰된 서버 내부 생성 파일은 `SYSTEM_GENERATED`로 구분한다.
- 격리 원본의 삭제 정책과 검사 이력·승인 증적의 보존 정책을 구분한다.

근거: [file_scan_job](../data-dictionary.md#table-file_scan_job), [file_asset](../data-dictionary.md#table-file_asset), [evidence_link](../data-dictionary.md#table-evidence_link).

### 5.2 공통 자료도 기본적으로 각 회사 DB 안에서 연결한다

| 데이터 | A제약에서 사용하는 예 | 권고하는 저장·참조 범위 |
|---|---|---|
| 역할·활동·선후행 조건 | A제약의 검토자 역할과 검증 활동 목록 | A제약 DB. 프로젝트 생성 당시 조건 사본 유지 |
| 라이브러리 | A제약에서 선택한 표준 시험 항목 | A제약 DB에 적재한 원본을 참조 |
| 규정·내부 SOP | 공개 규정 판본과 A제약 내부 SOP | 사용한 문서판·조항을 A제약 DB에서 보존 |
| Workflow·서명·감사기록 | A제약 프로젝트의 승인과 변경 기록 | 대상 업무와 같은 A제약 DB |
| 파일 메타데이터·검사 이력 | A제약 증적과 검사 결과 | A제약 DB. 실제 파일 저장소도 회사별 접근 제한 |
| AI 작업·결과 | A제약 자료를 이용한 초안 생성 | A제약 DB. 공용 Worker에도 고객사 정보 유지 |

표준 라이브러리나 공개 규정을 중앙에서 만들어 각 회사에 배포하는 것은 가능하다. 각 DB에 복사해 사용하는 방식과 중앙 DB를 업무에서 직접 참조하는 방식은 다르다. 기본안은 **각 고객사 DB에 필요한 자료를 적재하고 기존 FK를 유지하는 것**이며, 배포로 고객사의 수정 내용이나 과거 승인 근거를 덮어쓰지 않는다.

현재 73개 테이블에는 RAG 문서 청크·임베딩 저장 구조가 없다. AI 검색 저장소를 도입하면 고객사별 검색 범위를 별도로 설계해야 한다.

## 6. 담당자와 확인할 사항

| 확인할 내용 | 담당 | 확인할 결과 |
|---|---|---|
| 본사·공장·연구소 구분이 실제로 필요한가 | 업무 담당·DB 담당 | `organization`의 내부 조직 용도 확정 |
| 소속 표시 외에 사업장별 조회 제한·조직 계층이 필요한가 | 업무 담당·BE·DB 담당 | 추가 권한·조직 구조의 필요 여부 |
| 로그인한 회사와 연결할 DB를 어떻게 결정하는가 | BE·인프라 | 인증된 고객사와 전용 DB의 연결 방식 |
| 신규 회사 DB를 어떻게 준비하는가 | 인프라·BE·DB 담당 | 전용 환경, 73개 테이블, 기본 조직·역할·기준정보 초기화 |
| 파일 검사·AI 작업이 어느 회사 작업인지 어떻게 확인하는가 | BE·인프라·AI | 비동기 처리 중 고객사·작업·원본 검증 |
| 회사별 파일·검색·백업·DB 변경 적용을 어떻게 관리하는가 | 인프라·BE | 회사별 접근·복구·변경 적용 범위 |

접속할 회사의 DB 주소·자격증명 등 운영 정보는 인증·접속 설정 또는 별도 운영 관리 영역에서 관리한다. 이를 위해 73개 업무 테이블에 고객사 ID를 일괄 추가할 필요는 없다. 업로더·프로젝트가 NULL일 수 있는 시스템 생성 파일이나 자동 작업에서도 고객사를 확인할 수 있어야 한다.

[DB 기술 제안서](./DATABASE_TECHNOLOGY_PROPOSAL.md)의 제품·배포 선택지와 함께 검토한다. ERD용 SQL 14개를 합쳐 실행하는 것은 고객사 DB 초기화 절차가 아니며, 운영용 초기화·변경 SQL은 별도로 구성한다.

## 7. 검증 범위와 확인 기준

### 명세에서 확인한 구조

| 항목 | 확인 결과 |
|---|---|
| 테이블·컬럼 수 | 73개·1,177개 |
| ERD 구성 | 14개, 각 그림 15개 이하, 전체 테이블의 배정 누락·중복 없음 |
| 조직 직접 참조 | `app_user`, `user_group`, `system_asset`의 3개 FK |
| 프로젝트의 시스템 소속 조회 | `validation_project.system_id → system_asset.system_id`, `system_asset.organization_id → organization.organization_id` |
| 본사 직원의 공장 프로젝트 참여 | `project_member`의 사용자·프로젝트·역할 연결로 표현 가능. 실제 접근·결재 권한은 별도 검증 |
| 조직 계층 | 부모 조직 컬럼 없음. 유형을 가진 평면 목록 |
| 파일 검사 | 고객사 DB·원본·판정·등록을 검증하는 업무 규칙 존재 |
| Silo 때문에 반드시 제거할 FK | 현재 검토 범위에서 확인되지 않음 |

### 구현 후 확인할 시나리오

| 확인할 상황 | 기대 결과 |
|---|---|
| A제약 로그인 상태에서 B제약의 ID를 전달 | B제약 DB로 전환되거나 데이터가 노출되지 않음 |
| A제약 본사 직원이 권한을 받아 공장 프로젝트에 참여 | 소속 조직이 달라도 필요한 참여·접근·결재 배정에 따라 허용 |
| A제약 공장 직원이 권한 없는 업무에 접근 | 같은 고객사라도 접근 차단 |
| 여러 회사의 AI·검사 작업이 동시에 실행 | DB 연결·검색·파일·작업 결과가 회사 간에 섞이지 않음 |
| 검사 결과에 다른 회사 작업 ID나 다른 원본 버전을 전달 | 잘못된 결과 반영·파일 등록 차단 |
| 검사 실패·검사 불가·위협 발견 파일을 사용 | 첨부·다운로드·미리보기·AI 입력 차단 유지 |
| 검사 결과가 중복 또는 늦게 도착 | 중복 파일 등록·이전 회차의 판정 변경 방지 |
| A제약 DB와 관련 파일을 복구하거나 DB 구조를 변경 | 대상 고객사와 적용 버전을 확인하고 B제약 데이터에 영향 없이 처리 |

위 시나리오는 구현·배포 후 검증할 기준이다. 데이터 예시와 FK의 정합성을 확인한 것이 실제 환경의 접근 차단을 검증했다는 뜻은 아니다.
