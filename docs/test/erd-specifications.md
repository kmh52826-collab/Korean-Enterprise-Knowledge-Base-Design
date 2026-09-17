<a id="top"></a>

# ERD Specifications

Validation Management Platform의 데이터 모델을 업무 영역별로 정리한 ERD 명세서입니다.

72개 테이블을 14개 주제로 구분하고, 각 영역의 관계 구조, 구조 개요, 테이블별 역할과 핵심 이해 포인트를 설명합니다. 각 ERD는 외부 참조를 포함해 15개 이하의 테이블로 구성하며, 아래 DrawSQL 링크에서 확인할 수 있습니다.

> [!NOTE]
> 이 문서는 데이터 모델링 및 시스템 설계를 설명하기 위한 자료입니다. 예시값은 비식별 샘플을 사용하며, 특정 조직의 실제 운영 데이터를 나타내지 않습니다.

> **전체 업무 흐름을 보려면**  
> [전체 구조 한눈에 보기](#overview)를 참고하세요.
>
> **상세 컬럼 정보가 필요한 경우**  
> 컬럼, 데이터 타입, PK·FK, NULL 허용 여부, 기본값, 제약조건과 업무 규칙은 [Data Dictionary](./data-dictionary.md)를 참고하세요.

## 목차

1. [조직·사용자·권한](#erd-01) — 10개 테이블
2. [시스템 인벤토리·프로젝트 관리](#erd-02) — 14개 테이블
3. [검증 계획·사전 평가](#erd-03) — 10개 테이블
4. [요구사항·설계·위험평가](#erd-04) — 15개 테이블
5. [IQ 설치 적격성 시험](#erd-05) — 13개 테이블
6. [OQ 운전 적격성 시험](#erd-06) — 13개 테이블
7. [PQ 성능 적격성 시험](#erd-07) — 13개 테이블
8. [일탈·조치·재수행](#erd-08) — 9개 테이블
9. [요구사항·시험 추적성](#erd-09) — 11개 테이블
10. [VSR 종합 보고](#erd-10) — 14개 테이블
11. [결재·전자서명·감사기록](#erd-11) — 15개 테이블
12. [산출물·문서 개정·파일](#erd-12) — 15개 테이블
13. [라이브러리·규정 근거](#erd-13) — 12개 테이블
14. [AI 생성·적용](#erd-14) — 12개 테이블

위 개수는 각 그림에 표시하는 주 소속과 외부 참조 테이블을 합한 수입니다. 주 소속 테이블은 72개로 중복 없이 배정하고, 공통 참조 테이블은 필요한 그림에 반복 표시합니다.

---

<a id="overview"></a>

## 전체 구조 한눈에 보기

```text
조직·사용자·권한 구성
    ↓
시스템 인벤토리 등록·개정·승인
    ↓
Validation 프로젝트 생성
    ├─ 검증 대상 승인 개정 채택
    └─ 참여자·수행 활동·선후행 조건 구성
    ↓
VP 검증 계획 · QIA 품질 영향 평가 · VA 공급업체 감사
    ↓
URS 요구사항 · F&DS 설계 문서(FDS·DDS)
    ↓
DQ 설계 적격성 평가 · FRA 기능 위험평가
    ↓
IQ · OQ · PQ 프로토콜 승인 및 시험 수행
    └─ FAIL → 일탈 조치·승인 → 재수행 또는 사유를 남긴 종료
    ↓
VSR 종합 보고 및 프로젝트 종료 요청·승인

업무 전반의 조회: 요구사항·설계·위험·시험 추적성 및 RTM 대시보드
공통 통제: 결재 · 전자서명 · 감사기록 · 문서 개정 · 파일·증적 보존
작성 지원: 라이브러리 · 규정 근거 · AI 초안 생성·선택·적용
```

위 도식은 업무 이해를 위한 흐름입니다. 실제 수행 범위는 `project_activity`의 선택 활동을 기준으로 하며, 선후행 조건은 프로젝트 생성 시 채택한 `validation_project.dependency_snapshot`으로 평가합니다. 전역 `activity_dependency`의 이후 변경은 기존 프로젝트에 자동 전파하지 않습니다.

FDS와 DDS는 F&DS 활동에 속하는 문서 종류입니다. RTM은 독립 수행·승인 단계가 아니라 추적 관계와 시험 결과를 조회하는 대시보드 기능이며, 프로젝트 진행률의 수행 활동 수에 포함하지 않습니다.

업무 항목의 개정, 시험 수행 회차·결과 정정, 산출물 문서 개정은 서로 구분합니다. 검증 대상 시스템의 당시 상태는 승인 원문과 `project_system_baseline`으로 식별하고, 시험 수행과 문서는 각각 채택한 기준을 보존합니다.

### 문서와 관계 구조 읽는 방법

| 구분 | 의미 |
|---|---|
| 주 소속 | 해당 ERD의 중심 테이블. 업무 데이터와 관계의 상세 구조를 표시합니다. |
| 외부 참조 | 다른 ERD가 주 소속인 테이블. 이 그림에서는 PK와 관계 설명에 필요한 컬럼을 표시합니다. |
| `A --FK--> B` | A의 외래키가 B의 PK를 참조하는 물리 관계입니다. 화살표는 업무 수행 순서를 의미하지 않습니다. |
| `A ..논리..> B` | 유형+ID 또는 JSON 안의 식별값으로 연결하는 논리 관계입니다. 물리 FK가 아닙니다. |

각 관계 구조는 핵심 연결을 요약합니다. 생성자·수정자 등 반복되는 사용자 참조는 도식에서 일부 생략하며, 정확한 컬럼과 전체 물리 FK는 Data Dictionary를 참고하세요. 역할 요약 표에는 해당 ERD에 표시한 테이블을 모두 포함합니다.

> [!IMPORTANT]
> 다형 참조와 JSON 내부 참조는 물리 FK 관계선으로 표현되지 않습니다. 이 문서의 논리 관계 도식과 설명을 함께 확인하세요. 대상 존재·프로젝트 소속·정확한 개정은 서버 업무 규칙으로 검증합니다.
>
> VSR·결재·문서 ERD는 여러 업무 원본 중 대표 대상을 표시합니다. 외부 참조의 일부 컬럼이 그림에 보이지 않는다고 해서 실제 테이블에 해당 컬럼이 없는 것은 아닙니다.

---

<a id="erd-01"></a>

## 1. 조직·사용자·권한

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/01-organization-security)

**주 소속 9개 · 외부 참조 1개 · 총 10개 테이블**

### 관계 구조

```text
app_user --FK--> organization
user_group --FK--> organization
user_role --FK--> app_user
user_role --FK--> role
user_group_member --FK--> user_group
user_group_member --FK--> app_user
group_role --FK--> user_group
group_role --FK--> role
group_role --FK--> validation_project          [PROJECT 범위]
access_permission_grant --FK--> app_user      [개인에게 부여]
access_permission_grant --FK--> user_group    [그룹에 부여]
access_permission_grant --FK--> validation_project [PROJECT 범위]
inventory_role_grant --FK--> app_user         [개인에게 부여]
inventory_role_grant --FK--> user_group       [그룹에 부여]
```

### 구조 개요

조직에 사용자와 그룹이 소속되고, 사용자는 여러 그룹에 참여할 수 있습니다. `user_role`은 개인의 업무 역할을, `group_role`은 그룹의 전역 또는 프로젝트 범위 역할을 관리합니다. 사용자 계정의 `permission_level`은 관리 권한등급이며 작성자·검토자·승인자 같은 업무 역할과 구분합니다.

화면의 조회·편집·폐기 허용은 `access_permission_grant`에서, 인벤토리의 작성·검토·승인·폐기 역할은 `inventory_role_grant`에서 관리합니다. 두 테이블은 개인 또는 그룹 중 한 곳에 권한을 부여합니다. `validation_project`는 프로젝트 범위의 역할·접근권한이 어느 프로젝트에 적용되는지 보여주는 외부 참조입니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `organization` | 사용자와 그룹이 소속되는 조직·고객사 정보 |
| 주 소속 | `app_user` | 사용자 계정, 소속, 관리 권한등급, 계정 상태 |
| 주 소속 | `role` | 작성자·검토자·승인자 등 업무 역할의 공통 정의 |
| 주 소속 | `user_role` | 사용자에게 직접 부여한 업무 역할 |
| 주 소속 | `user_group` | 조직별 사용자 그룹과 활성 상태 |
| 주 소속 | `user_group_member` | 사용자의 그룹 소속과 참여·탈퇴 상태 |
| 주 소속 | `group_role` | 그룹에 부여한 전역·프로젝트 범위 업무 역할 |
| 주 소속 | `access_permission_grant` | 개인·그룹별 메뉴 또는 프로젝트 조회·편집·폐기 권한 |
| 주 소속 | `inventory_role_grant` | 개인·그룹별 인벤토리 작성·검토·승인·폐기 역할 |
| 외부 참조 | `validation_project` | 프로젝트 범위 역할·접근권한의 적용 대상 |

### 핵심 이해 포인트

- 개인 권한과 활성 그룹·구성원을 통해 상속받은 권한을 기능별 OR로 합산합니다. 그룹 권한을 개인 직접 부여 행으로 복제하지 않습니다.
- 개인·그룹 중 하나만 지정하는 조건과 메뉴·프로젝트 범위의 컬럼 조합은 SQL의 `CHECK`로 표현합니다. 프로젝트 범위에서만 `project_id`를 사용합니다.
- 접근권한은 업무 역할을 자동 부여하지 않습니다. 편집·폐기에는 접근권한과 업무 역할이 모두 필요하며, 검토·승인은 Workflow 담당 또는 유효한 대체 배정도 확인합니다.
- 활성 그룹·구성원·접근권한·인벤토리 역할의 중복은 상태 조건을 포함해 판단합니다. SQL의 조건부 유일 인덱스와 명세의 업무 규칙을 함께 확인해야 합니다.
- 이 그림의 `inventory_role_grant`는 특정 시스템 행을 참조하지 않습니다. 인벤토리 업무 권한의 부여 구조를 설명합니다.

[↑ 목차로](#top)

---

<a id="erd-02"></a>

## 2. 시스템 인벤토리·프로젝트 관리

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/02-inventory-project)

**주 소속 9개 · 외부 참조 5개 · 총 14개 테이블**

### 관계 구조

```text
system_asset --FK--> organization
system_asset_revision --FK--> system_asset
system_asset_revision --FK--> electronic_signature
validation_project --FK--> system_asset
project_system_baseline --FK--> validation_project
project_system_baseline --FK--> system_asset_revision [채택한 승인 이벤트]
validation_project --FK--> project_system_baseline   [현재 기준]
project_member --FK--> validation_project
project_member --FK--> app_user
project_member --FK--> role
project_activity --FK--> validation_project
project_activity --FK--> validation_activity
activity_dependency --FK--> validation_activity     [선행·후행 활동]
project_closure_request --FK--> validation_project
project_closure_request --FK--> workflow_instance
validation_project --FK--> project_closure_request   [현재 종료 요청]
validation_project.dependency_snapshot ..논리..> activity_dependency
workflow_instance ..논리..> system_asset            [대상 유형·ID·개정]
workflow_instance ..논리..> project_closure_request  [대상 유형·ID·요청 버전]
electronic_signature ..논리..> system_asset         [대상 유형·ID·개정]
electronic_signature ..논리..> project_closure_request [대상 유형·ID·요청 버전]
```

### 구조 개요

시스템 인벤토리의 현재 정보는 `system_asset`에, 처리 이벤트와 당시 전체 원문은 `system_asset_revision`에 저장합니다. 프로젝트는 검증 대상 시스템을 지정하고, `project_system_baseline`을 통해 실제로 채택한 승인 이벤트를 식별합니다. 인벤토리가 이후 개정되어도 과거 프로젝트 기준이 가리키는 승인 원문은 유지됩니다.

프로젝트 참여자는 사용자·역할로 관리하며, 수행 활동은 공통 활동 마스터에서 선택합니다. 전역 선후행 조건은 프로젝트 생성 시 적용 버전과 원문을 복사해 보존합니다. 종료 요청은 프로젝트와 별도 행으로 관리하여 정상·강제 종료의 상신, 반려, 재요청 및 결재 이력을 구분합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `system_asset` | 시스템·장비의 현재 식별 정보, 분류, 승인·폐기 상태 |
| 주 소속 | `system_asset_revision` | 등록·저장·개정·승인·폐기 이벤트와 당시 전체 원문·해시 |
| 주 소속 | `validation_project` | 검증 프로젝트, 대상 시스템, 현재 기준·종료 요청, 적용 선후행 규칙 원문 |
| 주 소속 | `project_system_baseline` | 프로젝트가 채택한 인벤토리 승인 개정과 기준 변경 이력 |
| 주 소속 | `project_member` | 프로젝트별 참여 사용자와 업무 역할 |
| 주 소속 | `validation_activity` | VP·QIA·URS·시험 등 수행 활동의 공통 정의와 표시 순서 |
| 주 소속 | `project_activity` | 프로젝트별 활동 선택 여부, 필수 여부 및 진행 상태 |
| 주 소속 | `activity_dependency` | 활동 활성화·완료를 판단하는 전역 선후행 조건 |
| 주 소속 | `project_closure_request` | 종료 요청 회차, 종료 유형·사유, 당시 진행률과 결재 연결 |
| 외부 참조 | `organization` | 시스템 및 사용자의 소속 조직 |
| 외부 참조 | `app_user` | 프로젝트 참여자, 기준 채택자, 각 업무 처리자 |
| 외부 참조 | `role` | 프로젝트 참여자에게 부여하는 업무 역할 |
| 외부 참조 | `workflow_instance` | 인벤토리 및 프로젝트 종료의 검토·승인 절차 |
| 외부 참조 | `electronic_signature` | 인벤토리 승인·폐기 및 종료 대상에 대한 전자서명 |

### 핵심 이해 포인트

- 소프트웨어 버전, 인벤토리 개정번호, 프로젝트 기준 순번은 서로 다른 값입니다. 프로젝트 기준은 승인 원문을 가진 `APPROVE` 이벤트를 채택합니다.
- 같은 시스템·개정에 여러 저장·처리 이벤트가 존재할 수 있습니다. 전체 이벤트를 하나로 제한하지 않으며, 최종 승인 이벤트의 유일성과 원문 보존 규칙을 별도로 적용합니다.
- 현재 기준과 현재 종료 요청을 가리키는 FK는 조회용 참조입니다. 해당 행이 같은 프로젝트에 속하는지, 기준이 현재 유효한 승인본인지 등은 FK 존재 확인 외에 업무 검증이 필요합니다.
- `dependency_snapshot`은 채택 당시 조건과 판정 정의를 보존합니다. 전역 `activity_dependency`의 변경이 기존 프로젝트의 적용 원문을 자동으로 바꾸지 않습니다.
- RTM은 수행 활동으로 선택하지 않습니다. 프로젝트 진행률·완료 판정은 실제 선택 활동과 해당 조건에 따라 계산하며, 종료 요청 당시의 진행률은 별도로 보존합니다.

[↑ 목차로](#top)

---

<a id="erd-03"></a>

## 3. 검증 계획·사전 평가

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/03-planning-assessment)

**주 소속 6개 · 외부 참조 4개 · 총 10개 테이블**

### 관계 구조

```text
vp_plan --FK--> validation_project
vp_section --FK--> vp_plan
qia_assessment --FK--> validation_project
qia_module_item --FK--> qia_assessment
qia_process --FK--> qia_module_item
vendor_audit --FK--> validation_project
vendor_audit --FK--> app_user       [감사자 계정]
vendor_audit --FK--> file_asset     [평가 첨부파일]
vp_plan --FK--> workflow_instance
qia_module_item --FK--> workflow_instance
vendor_audit --FK--> workflow_instance
workflow_instance ..논리..> vp_plan         [대상 유형·ID·개정]
workflow_instance ..논리..> qia_module_item [대상 유형·ID·개정]
workflow_instance ..논리..> vendor_audit    [대상 유형·ID·개정]
```

### 구조 개요

검증 계획(VP), 품질 영향 평가(QIA), 공급업체 감사(VA)를 프로젝트별로 관리합니다. VP는 계획 개정과 그 개정에 포함된 목차·본문으로 구성됩니다. QIA는 프로젝트의 공통 Part 11 평가, 개정별 모듈 평가, 해당 모듈에 속하는 프로세스 평가로 나뉩니다.

공급업체 감사는 감사자 계정·당시 표시명, 감사일, 첨부파일과 개정을 보존합니다. VP, QIA 모듈, VA의 승인·폐기는 각각 Workflow에 연결합니다. QIA의 질문과 판정 규칙은 버전 이름뿐 아니라 실제 정의를 원문으로 함께 저장하여 당시 응답의 해석 기준을 유지합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `vp_plan` | 프로젝트 검증 계획의 논리 식별자·개정·승인 상태 |
| 주 소속 | `vp_section` | 특정 VP 개정의 목차, 본문, 표시 순서와 포함 여부 |
| 주 소속 | `qia_assessment` | 프로젝트 공통 Part 11 응답·결론과 질문·판정 정의 |
| 주 소속 | `qia_module_item` | QIA 모듈별 개정·집계 판정·승인·폐기 상태 |
| 주 소속 | `qia_process` | 모듈 개정에 속한 프로세스의 GxP 응답과 질문·판정 정의 |
| 주 소속 | `vendor_audit` | 공급업체 감사 개정, 감사자·일자와 평가 첨부파일 |
| 외부 참조 | `validation_project` | 계획과 사전 평가가 속하는 프로젝트 |
| 외부 참조 | `app_user` | 작성·수정자, 감사자 및 결재 상신자 |
| 외부 참조 | `workflow_instance` | 계획·모듈·감사 개정의 업무 승인 및 폐기 절차 |
| 외부 참조 | `file_asset` | 공급업체 감사에 연결되는 첨부파일 식별 정보 |

### 핵심 이해 포인트

- 프로젝트당 VP 논리 항목은 하나이고 개정은 여러 개입니다. 목차는 특정 `vp_id`에 속하며, 같은 개정 내 목차 키와 표시 순서는 각각 중복될 수 없습니다.
- QIA 공통 평가는 프로젝트당 하나이지만, 모듈은 여러 개이며 모듈별 개정·승인을 관리합니다. 프로세스는 특정 모듈 개정에 속하므로 서로 다른 개정의 응답을 섞지 않습니다.
- Part 11 결과와 모듈 GxP 결과는 응답에서 계산하는 값입니다. 질문·규칙 버전과 함께 `question_set_snapshot`, `rule_snapshot`의 실제 정의를 보존합니다.
- 프로세스가 없는 모듈의 판정은 `NON_GXP`이며, 프로세스 존재 여부 자체는 승인 제한 조건이 아닙니다. 이 표시가 실제 프로세스 평가 완료를 의미하지는 않습니다.
- `is_current_version`은 최신 개정 여부이며 최신 승인본 여부와 다릅니다. 개정행 PK, 논리키, 승인번호를 구분하고 승인된 개정의 본문·하위 응답은 보존합니다.

[↑ 목차로](#top)

---

<a id="erd-04"></a>

## 4. 요구사항·설계·위험평가

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/04-requirements-design-risk)

**주 소속 7개 · 외부 참조 8개 · 총 15개 테이블**

### 관계 구조

```text
requirement --FK--> validation_project
fds_spec --FK--> validation_project
dds_spec --FK--> validation_project
dq_assessment --FK--> validation_project
fra_assessment --FK--> validation_project
dq_item --FK--> dq_assessment
dq_item --FK--> requirement   [평가한 URS 개정]
dq_item --FK--> fds_spec      [선택한 FDS 승인 개정]
dq_item --FK--> dds_spec      [선택한 DDS 승인 개정]
fra_item --FK--> fra_assessment
fra_item --FK--> requirement  [위험평가 대상 URS 개정]
fra_item --FK--> regulatory_clause [내부 SOP 조항]
requirement_regulation --FK--> requirement
requirement_regulation --FK--> regulatory_clause
regulatory_clause --FK--> regulatory_source
regulatory_source --FK--> file_asset
fds_spec --FK--> file_asset
dds_spec --FK--> file_asset
requirement --FK--> library_item [복사 출처]
fra_item --FK--> library_item    [복사 출처]
requirement --FK--> workflow_instance
fds_spec --FK--> workflow_instance
dds_spec --FK--> workflow_instance
dq_item --FK--> workflow_instance
fra_item --FK--> workflow_instance
workflow_instance ..논리..> requirement / fds_spec / dds_spec / dq_item / fra_item
                           [target_table_name으로 결정되는 승인 대상·개정]
```

### 구조 개요

URS는 검증할 요구사항을, FDS·DDS는 업로드한 기능·상세 설계 문서의 개정을 관리합니다. DQ는 특정 URS 개정과 선택한 FDS·DDS 승인 개정을 묶어 설계 적합성 판정을 기록합니다. FRA는 특정 요구사항의 위험 시나리오, 심각도·발생가능성·검출도, 적용 판정 규칙과 승인 당시 결과를 관리합니다.

규정 근거는 문서·판본을 나타내는 `regulatory_source`와 조항인 `regulatory_clause`로 구분합니다. URS와 규정 조항은 `requirement_regulation`으로 연결하고, FRA는 내부 SOP 조항을 직접 참조합니다. 라이브러리는 요구사항·위험 항목의 복사 출처이며, 파일과 Workflow는 설계 원본 및 승인·폐기 절차를 연결합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `requirement` | URS 항목의 상세 요구사항·수용 기준·출처와 개정·승인 상태 |
| 주 소속 | `fds_spec` | 기능 설계 문서의 개정, 업로드 파일, 승인·폐기 상태 |
| 주 소속 | `dds_spec` | 상세 설계 문서의 개정, 업로드 파일, 승인·폐기 상태 |
| 주 소속 | `dq_assessment` | 프로젝트별 설계 적격성 평가의 상위 정보 |
| 주 소속 | `dq_item` | URS·FDS·DDS의 특정 개정에 대한 적합성 판정과 검토 내용 |
| 주 소속 | `fra_assessment` | 프로젝트별 기능 위험평가의 상위 정보 |
| 주 소속 | `fra_item` | 요구사항별 위험 시나리오·평가 입력·규칙 원문·승인 결과·SOP 근거 |
| 외부 참조 | `validation_project` | 요구사항·설계·평가가 속하는 프로젝트 |
| 외부 참조 | `app_user` | 작성·수정자, 설계 파일 업로더와 DQ 판정 수행자 |
| 외부 참조 | `workflow_instance` | 업무 항목 개정의 승인·폐기 절차 |
| 외부 참조 | `library_item` | URS·FRA 초안의 재사용 원본 |
| 외부 참조 | `file_asset` | FDS·DDS 및 규정 원문에 연결되는 파일 |
| 외부 참조 | `regulatory_clause` | URS 규정 근거 및 FRA의 내부 SOP 근거 조항 |
| 외부 참조 | `requirement_regulation` | URS 개정과 규정 조항 사이의 연결 |
| 외부 참조 | `regulatory_source` | 규정·가이드·내부 SOP의 출처 문서와 판본 |

### 핵심 이해 포인트

- 논리키는 같은 항목의 개정들을 묶고, PK는 특정 개정행을 식별합니다. DQ·FRA의 URS 참조와 DQ의 설계 참조는 정확한 개정행을 가리키며 새 개정으로 자동 교체하지 않습니다.
- DQ 항목은 URS 한 개정과 FDS·DDS를 각각 최대 한 개정 연결합니다. 승인 요청에는 같은 프로젝트의 유효 승인 FDS·DDS 중 하나 이상, 판정 결과와 수행자·시각이 필요합니다. 이 조건은 FK 존재 여부 외에 업무 검증으로 확인합니다.
- FDS와 DDS는 서로 독립된 설계 문서입니다. URS와 FDS·DDS 사이에 직접 FK가 있는 것으로 읽지 않으며, 이 그림에서는 DQ 항목의 참조를 통해 평가 근거가 연결됩니다.
- FRA의 위험 결과는 평가 입력과 `rule_snapshot`의 정의로 계산하고 승인 당시 `risk_result_snapshot`을 보존합니다. `sop_clause_id`는 내부 SOP 유형의 조항만 허용합니다.
- 파일을 교체하면 새 설계 개정과 새 파일 ID를 생성합니다. 라이브러리에서 가져온 초안도 독립된 업무 항목으로 관리하므로 출처 수정이 승인 원문에 자동 반영되지 않습니다.

[↑ 목차로](#top)

---

<a id="erd-05"></a>

## 5. IQ 설치 적격성 시험

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/05-iq-testing)

**주 소속 5개 · 외부 참조 8개 · 총 13개 테이블**

### 관계 구조

```text
iq_assessment.project_id --FK--> validation_project.project_id
iq_item.iq_id --FK--> iq_assessment.iq_id                 [최초 등록 상위]
iq_assessment.item_revision_refs ..논리..> iq_item       [개정별 실제 구성]
iq_step.iq_item_id --FK--> iq_item.iq_item_id
iq_execution.iq_item_id --FK--> iq_item.iq_item_id
iq_step_execution.execution_id --FK--> iq_execution.execution_id
iq_step_execution.step_id --FK--> iq_step.step_id
iq_execution.system_baseline_id --FK--> project_system_baseline.baseline_id
project_system_baseline.project_id --FK--> validation_project.project_id
iq_item.source_library_id --FK--> library_item.library_id
iq_item.protocol_workflow_id --FK--> workflow_instance.workflow_instance_id
iq_execution.result_workflow_id --FK--> workflow_instance.workflow_instance_id
iq_execution.execution_signature_id --FK--> electronic_signature.signature_id
iq_execution.executed_by --FK--> app_user.user_id
evidence_link.file_id --FK--> file_asset.file_id
evidence_link.target_entity_id ..논리..> iq_item / iq_execution / iq_step_execution
```

### 구조 개요

IQ는 평가 문서, 시험 프로토콜, 세부 절차, 실제 수행, 절차별 완료 기록으로 나뉩니다. `iq_assessment`는 평가 범위와 시험 구성을 관리하고, `iq_item`·`iq_step`은 승인할 시험 내용을 관리합니다. `iq_execution`·`iq_step_execution`은 특정 프로토콜 개정을 실제로 수행한 결과를 보존합니다.

`iq_item.iq_id`는 시험을 최초 등록한 평가 개정을 가리킵니다. 후속 평가 개정에 포함되는 실제 시험 목록과 순서는 `item_revision_refs`에 고정하므로, 내용이 바뀌지 않은 시험 개정은 여러 평가 개정에서 재사용할 수 있습니다. 수행 당시의 검증 대상 기준은 `project_system_baseline`에 연결합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `iq_assessment` | IQ 평가 개정, 시험 구성, 프로토콜·결과 승인 상태 요약 |
| 주 소속 | `iq_item` | 설치 적격성 시험의 프로토콜 개정과 승인·폐기 정보 |
| 주 소속 | `iq_step` | 특정 시험 개정에 속하는 순서별 세부 절차 |
| 주 소속 | `iq_execution` | 수행 회차·결과 정정, 판정, 수행 서명, 적용한 검증 대상 기준 |
| 주 소속 | `iq_step_execution` | 한 수행에서 각 절차의 완료 여부·처리자·시각 |
| 외부 참조 | `validation_project` | IQ 평가가 속하는 프로젝트 |
| 외부 참조 | `project_system_baseline` | 수행 당시 프로젝트가 채택한 검증 대상 기준 |
| 외부 참조 | `app_user` | 작성자·수행자·절차 완료 처리자 등 사용자 식별 |
| 외부 참조 | `library_item` | 시험을 복사해 등록한 라이브러리 원본 |
| 외부 참조 | `workflow_instance` | 프로토콜 승인과 수행 결과 승인 절차 |
| 외부 참조 | `electronic_signature` | 수행 결과 등록 및 시험 폐기에 연결된 서명 |
| 외부 참조 | `evidence_link` | 시험 항목·수행·절차 수행과 증적 파일의 연결 |
| 외부 참조 | `file_asset` | 증적 파일 식별과 원본 내용 해시 |

### 핵심 이해 포인트

- **평가 구성과 최초 상위를 구분합니다.** 현재 평가의 시험 목록을 `iq_id`만으로 조회하지 않고 해당 평가의 `item_revision_refs`를 사용합니다. 구성에 같은 논리 시험의 여러 개정을 중복 포함하지 않습니다.
- **개정 대상별 변경 범위가 다릅니다.** 출력 문서의 제목·본문만 변경하면 [12번 산출물 ERD](#erd-12)의 문서 개정을 생성합니다. 평가 범위·시험 구성이 바뀌면 평가 개정을, 시험 본문·절차가 바뀌면 해당 시험 개정을 생성합니다.
- **프로토콜 승인, 수행 판정, 결과 승인은 별개입니다.** 승인된 프로토콜에서 수행하며 `PASS`·`FAIL`·`NA` 판정만으로 결과 승인이 완료되지는 않습니다. 상위 평가의 상태는 구성 항목과 수행에서 산출한 요약입니다.
- **재수행과 정정을 분리합니다.** 재수행은 `attempt_no`, 같은 회차의 결과 정정은 `record_revision`을 증가시킵니다. 정정 결과는 최초 수행의 기준을 유지하고, 재수행은 새 수행 당시의 기준을 연결합니다.
- **연결 대상의 개정이 일치해야 합니다.** 절차 완료 기록은 수행과 같은 프로토콜 개정의 절차만 참조합니다. 증적은 유형+ID로 연결하며, URS 연결은 [9번 추적성 ERD](#erd-09), 실패 후 처리는 [8번 일탈 ERD](#erd-08)에서 설명합니다.

[↑ 목차로](#top)

---

<a id="erd-06"></a>

## 6. OQ 운전 적격성 시험

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/06-oq-testing)

**주 소속 5개 · 외부 참조 8개 · 총 13개 테이블**

### 관계 구조

```text
oq_assessment.project_id --FK--> validation_project.project_id
oq_item.oq_id --FK--> oq_assessment.oq_id                 [최초 등록 상위]
oq_assessment.item_revision_refs ..논리..> oq_item       [개정별 실제 구성]
oq_step.oq_item_id --FK--> oq_item.oq_item_id
oq_execution.oq_item_id --FK--> oq_item.oq_item_id
oq_step_execution.execution_id --FK--> oq_execution.execution_id
oq_step_execution.step_id --FK--> oq_step.step_id
oq_execution.system_baseline_id --FK--> project_system_baseline.baseline_id
project_system_baseline.project_id --FK--> validation_project.project_id
oq_item.source_library_id --FK--> library_item.library_id
oq_item.protocol_workflow_id --FK--> workflow_instance.workflow_instance_id
oq_execution.result_workflow_id --FK--> workflow_instance.workflow_instance_id
oq_execution.execution_signature_id --FK--> electronic_signature.signature_id
oq_execution.executed_by --FK--> app_user.user_id
evidence_link.file_id --FK--> file_asset.file_id
evidence_link.target_entity_id ..논리..> oq_item / oq_execution / oq_step_execution
```

### 구조 개요

OQ는 운전 적격성을 평가하는 시험 프로토콜과 실제 수행 결과를 관리합니다. `oq_assessment`가 평가 개정과 시험 구성을 관리하고, `oq_item`·`oq_step`이 검증 내용·기대 결과·허용 기준과 세부 절차를 정의합니다. 수행 결과와 절차별 완료 기록은 별도 테이블에 저장합니다.

평가 개정별 구성은 `item_revision_refs`에 정확한 시험 PK·버전·순서로 기록합니다. `oq_item.oq_id`는 최초 등록 상위이므로, 기존 시험이 새 평가 개정에 포함되더라도 그 FK를 옮기지 않습니다. 전체 구조와 개정 원칙은 IQ와 같으며 OQ 데이터와 승인 상태는 독립적으로 관리합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `oq_assessment` | OQ 평가 개정, 시험 구성, 프로토콜·결과 승인 상태 요약 |
| 주 소속 | `oq_item` | 운전 적격성 시험의 프로토콜 개정과 승인·폐기 정보 |
| 주 소속 | `oq_step` | 특정 OQ 시험 개정에 속하는 세부 절차 |
| 주 소속 | `oq_execution` | 회차별 실제 결과와 판정, 결과 정정·승인, 수행 기준 |
| 주 소속 | `oq_step_execution` | 수행별 절차 완료 여부와 처리 기록 |
| 외부 참조 | `validation_project` | OQ 평가가 속하는 프로젝트 |
| 외부 참조 | `project_system_baseline` | 해당 OQ 수행에 고정된 검증 대상 기준 |
| 외부 참조 | `app_user` | 시험 작성·수행·확인에 참여한 사용자 |
| 외부 참조 | `library_item` | 복사해 등록한 OQ 시험의 원본 라이브러리 |
| 외부 참조 | `workflow_instance` | 프로토콜 및 결과의 개별 승인 절차 |
| 외부 참조 | `electronic_signature` | 수행 결과 등록과 폐기 처리의 서명 |
| 외부 참조 | `evidence_link` | OQ 시험·수행·절차 수행에 대한 증적 연결 |
| 외부 참조 | `file_asset` | 증적 파일의 식별 정보와 내용 해시 |

### 핵심 이해 포인트

- **승인된 프로토콜과 절차는 보존합니다.** 시험 내용 변경은 같은 `item_key`의 새 개정으로 기록하고, 기존 수행은 원래 프로토콜 개정을 계속 참조합니다.
- **평가 개정과 문서 개정을 구분합니다.** 시험 구성 변경은 평가의 `item_revision_refs`에 고정합니다. 출력 제목·본문만 바뀌는 경우에는 [12번 산출물 ERD](#erd-12)의 문서 개정만 생성할 수 있습니다.
- **판정과 승인 상태를 따로 읽습니다.** `qualification_result`는 수행 판정이고 `record_status`는 결과 승인 상태입니다. 평가 헤더의 승인 요약으로 개별 시험·수행 상태를 덮어쓰지 않습니다.
- **수행 회차와 결과 정정은 다른 이력입니다.** 재수행은 새 `attempt_no`, 결과 정정은 같은 회차의 새 `record_revision`으로 남깁니다. 과거 결과·절차 기록·증적·서명은 유지합니다.
- **IQ·PQ와의 선후행은 시험 테이블끼리 직접 FK로 연결하지 않습니다.** 프로젝트 활동의 선후행 조건은 [2번 프로젝트 ERD](#erd-02), URS 추적성은 [9번 ERD](#erd-09), 실패 조치는 [8번 ERD](#erd-08)에서 다룹니다.

[↑ 목차로](#top)

---

<a id="erd-07"></a>

## 7. PQ 성능 적격성 시험

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/07-pq-testing)

**주 소속 5개 · 외부 참조 8개 · 총 13개 테이블**

### 관계 구조

```text
pq_assessment.project_id --FK--> validation_project.project_id
pq_item.pq_id --FK--> pq_assessment.pq_id                 [최초 등록 상위]
pq_assessment.item_revision_refs ..논리..> pq_item       [개정별 실제 구성]
pq_step.pq_item_id --FK--> pq_item.pq_item_id
pq_execution.pq_item_id --FK--> pq_item.pq_item_id
pq_step_execution.execution_id --FK--> pq_execution.execution_id
pq_step_execution.step_id --FK--> pq_step.step_id
pq_execution.system_baseline_id --FK--> project_system_baseline.baseline_id
project_system_baseline.project_id --FK--> validation_project.project_id
pq_item.source_library_id --FK--> library_item.library_id
pq_item.protocol_workflow_id --FK--> workflow_instance.workflow_instance_id
pq_execution.result_workflow_id --FK--> workflow_instance.workflow_instance_id
pq_execution.execution_signature_id --FK--> electronic_signature.signature_id
pq_execution.executed_by --FK--> app_user.user_id
evidence_link.file_id --FK--> file_asset.file_id
evidence_link.target_entity_id ..논리..> pq_item / pq_execution / pq_step_execution
```

### 구조 개요

PQ는 성능 적격성 시험의 정의와 수행 결과를 구분해 관리합니다. 평가 개정에 포함되는 시험 목록은 `pq_assessment`에, 시험 본문과 허용 기준은 `pq_item`에, 순서별 절차는 `pq_step`에 저장합니다. 실제 판정과 절차 완료 기록은 각각 `pq_execution`과 `pq_step_execution`에 남깁니다.

IQ·OQ와 같은 구조를 사용하되 PQ의 평가·시험·수행 기록은 별도 테이블로 관리합니다. 평가 구성에 기존 시험 개정을 다시 포함할 수 있으므로, 평가 문서가 바뀌었다는 이유만으로 변경 없는 시험과 과거 결과를 복제할 필요가 없습니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `pq_assessment` | PQ 평가 개정, 시험 구성, 프로토콜·결과 승인 상태 요약 |
| 주 소속 | `pq_item` | 성능 적격성 시험 내용·기대 결과·허용 기준의 개정 |
| 주 소속 | `pq_step` | PQ 프로토콜 개정에 속하는 순서별 절차 |
| 주 소속 | `pq_execution` | 실제 수행 회차·결과 정정·판정·서명·승인 기록 |
| 주 소속 | `pq_step_execution` | 특정 수행에서 각 절차를 완료한 기록 |
| 외부 참조 | `validation_project` | PQ 평가의 프로젝트 범위 |
| 외부 참조 | `project_system_baseline` | 수행 시 채택한 검증 대상 기준 |
| 외부 참조 | `app_user` | 시험 작성자·수행자·확인자 등의 사용자 |
| 외부 참조 | `library_item` | PQ 시험을 가져온 라이브러리 원본 |
| 외부 참조 | `workflow_instance` | 프로토콜 승인과 결과 승인 경로 |
| 외부 참조 | `electronic_signature` | 수행 결과 등록 및 폐기 서명 |
| 외부 참조 | `evidence_link` | PQ 시험·수행·절차 수행과 증적의 연결 |
| 외부 참조 | `file_asset` | 증적 파일 식별과 내용 해시 |

### 핵심 이해 포인트

- **평가 구성은 개정별 전체 목록입니다.** `item_revision_refs`의 항목 PK·버전·순서로 구성을 판단합니다. `pq_item.pq_id`는 최초 등록 상위이며 현재 구성 여부를 대신하지 않습니다.
- **프로토콜 개정과 결과 이력은 독립적입니다.** 같은 승인 프로토콜에 여러 수행 회차가 생길 수 있고, 한 수행 회차에도 결과 정정 이력이 생길 수 있습니다. 각각 `revision_number`, `attempt_no`, `record_revision`으로 식별합니다.
- **결과 등록 전에 절차 완료와 서명이 필요합니다.** 결과 등록 이후에는 절차 완료 상태와 증적을 잠그며, 수정이 필요하면 기존 기록을 보존하는 정정 절차를 따릅니다.
- **시험 대상은 수행 당시 기준으로 고정합니다.** 프로젝트의 현재 기준이 나중에 바뀌어도 과거 `pq_execution.system_baseline_id`를 덮어쓰지 않습니다.
- **시험의 역할별 연결을 구분합니다.** URS와의 관계는 [9번 추적성 ERD](#erd-09), 실패 이후 처리는 [8번 일탈 ERD](#erd-08), 승인된 결과의 종합 집계는 [10번 VSR ERD](#erd-10)에서 확인합니다.

[↑ 목차로](#top)

---

<a id="erd-08"></a>

## 8. 일탈·조치·재수행

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/08-deviation-rerun)

**주 소속 2개 · 외부 참조 7개 · 총 9개 테이블**

### 관계 구조

```text
deviation.project_id --FK--> validation_project.project_id
deviation_action_round.deviation_id --FK--> deviation.deviation_id
deviation.current_action_round_id --FK--> deviation_action_round.action_round_id
deviation_action_round.workflow_instance_id --FK--> workflow_instance.workflow_instance_id
deviation_action_round.signature_id --FK--> electronic_signature.signature_id
deviation.completion_signature_id --FK--> electronic_signature.signature_id
deviation.closure_signature_id --FK--> electronic_signature.signature_id
deviation.approved_by --FK--> app_user.user_id

deviation.source_entity_id ..논리..> iq_execution / oq_execution / pq_execution
deviation.rerun_execution_id ..논리..> iq_execution / oq_execution / pq_execution
deviation_action_round.failed_execution_id ..논리..> iq_execution / oq_execution / pq_execution
deviation_action_round.rerun_execution_id ..논리..> iq_execution / oq_execution / pq_execution
```

### 구조 개요

`deviation`은 최초 실패 수행, 일탈의 전체 진행 상태와 최종 종료 정보를 관리합니다. `deviation_action_round`는 실패에 대응한 조치 원문을 회차·개정별로 보존하고, 그 원문을 승인한 Workflow·전자서명 및 승인으로 허용한 재수행을 연결합니다.

실패 수행과 재수행은 `deviation.source_entity_type`에 따라 IQ·OQ·PQ 중 한 수행 테이블에서 식별합니다. 세 테이블을 동시에 참조하는 FK가 아니라 유형과 ID를 조합한 논리 참조입니다. 원본 실패는 그대로 유지하고, 재실패는 다음 조치 회차로 이어집니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `deviation` | 최초 실패, 현재 조치 요약, 최근 결과, 완료보고 또는 미수행 종료 정보 |
| 주 소속 | `deviation_action_round` | 실패별 조치 회차와 원문 개정, 승인·서명, 해당 승인으로 시작한 재수행 |
| 외부 참조 | `validation_project` | 일탈이 발생한 프로젝트 |
| 외부 참조 | `app_user` | 일탈 작성자·처리자·최종 승인자 |
| 외부 참조 | `workflow_instance` | 조치 원문 개정의 승인 절차 및 수행 결과 승인 절차 |
| 외부 참조 | `electronic_signature` | 조치 승인, 완료보고 승인, 미수행 종료의 서명 |
| 외부 참조 | `iq_execution` | IQ에서 발생한 실패 또는 재수행의 특정 결과 기록 |
| 외부 참조 | `oq_execution` | OQ에서 발생한 실패 또는 재수행의 특정 결과 기록 |
| 외부 참조 | `pq_execution` | PQ에서 발생한 실패 또는 재수행의 특정 결과 기록 |

### 핵심 이해 포인트

- **실패가 늘면 회차가, 같은 실패의 원문을 고치면 개정이 증가합니다.** 재수행 실패는 다음 `round_number`를 생성하고, 반려 후 같은 조치를 수정·재상신하면 `revision_number`를 증가시킵니다.
- **서명은 특정 조치 원문을 대상으로 합니다.** `action_round_id`와 `ACTION-{round_number}-{revision_number}`로 대상을 식별하며, 같은 회차의 승인된 개정은 최대 한 건입니다.
- **최초 실패와 최근 상태를 구분합니다.** `deviation.source_entity_id`는 최초 실패를 유지합니다. 현재 조치·승인 정보는 `current_action_round_id`의 요약이며 과거 원문은 회차 테이블에서 조회합니다.
- **재수행 연결은 승인 근거까지 이어져야 합니다.** 조치 회차는 그 승인으로 시작한 재수행을 기록하고, 다음 실패 회차는 직전 재수행 또는 그 회차의 정정 결과를 참조합니다. 같은 프로젝트·단계·논리 시험인지 서버에서 검증합니다.
- **두 종료 경로를 구분합니다.** 재수행 성공 후 완료보고 승인과, 승인된 조치 이후 재수행 없이 사유를 입력하는 종료는 서로 다른 서명 필드를 사용합니다. 종료 시 두 최종 서명 중 정확히 하나를 보존합니다.

[↑ 목차로](#top)

---

<a id="erd-09"></a>

## 9. 요구사항·시험 추적성

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/09-traceability)

**주 소속 1개 · 외부 참조 10개 · 총 11개 테이블**

### 관계 구조

```text
traceability_link.project_id --FK--> validation_project.project_id
traceability_link.created_by / updated_by --FK--> app_user.user_id

traceability_link.source_entity_id / target_entity_id ..논리..> requirement
traceability_link.source_entity_id / target_entity_id ..논리..> fds_spec / dds_spec
traceability_link.source_entity_id / target_entity_id ..논리..> dq_item / fra_item
traceability_link.source_entity_id / target_entity_id ..논리..> iq_item / oq_item / pq_item

dq_item.requirement_id --FK--> requirement.requirement_id
dq_item.fds_revision_id --FK--> fds_spec.fds_id
dq_item.dds_revision_id --FK--> dds_spec.dds_id
fra_item.requirement_id --FK--> requirement.requirement_id
```

### 구조 개요

`traceability_link`는 요구사항·설계·설계 적격성·위험평가·시험 항목의 특정 개정 사이 관계를 저장합니다. 출발 유형·ID와 도착 유형·ID를 `link_type`과 함께 기록하여 구현·평가·검증·위험 완화 관계를 표현합니다. 같은 시험에 여러 URS를 연결하는 관계도 이 테이블로 관리합니다.

대시보드의 RTM은 이 연결과 현재 유효한 원본·수행 상태를 조회해 구성합니다. 별도 수행 단계나 RTM 전용 승인 문서를 만들지 않습니다. 그림에 함께 표시한 DQ·FRA의 FK는 각 업무가 직접 참조하는 근거이며, 공통 추적 관계의 다형 참조와 선 종류를 구분합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `traceability_link` | 프로젝트 내 업무 개정 간 추적 관계와 연결 근거 |
| 외부 참조 | `validation_project` | 추적 관계의 프로젝트 범위 |
| 외부 참조 | `app_user` | 관계를 작성·수정한 사용자 |
| 외부 참조 | `requirement` | 추적의 기준이 되는 URS 개정 |
| 외부 참조 | `fds_spec` | 기능 설계 문서의 특정 개정 |
| 외부 참조 | `dds_spec` | 상세 설계 문서의 특정 개정 |
| 외부 참조 | `dq_item` | URS와 설계 근거를 평가한 DQ 항목 개정 |
| 외부 참조 | `fra_item` | 요구사항의 위험을 평가한 FRA 항목 개정 |
| 외부 참조 | `iq_item` | 요구사항을 검증하는 IQ 프로토콜 개정 |
| 외부 참조 | `oq_item` | 요구사항을 검증하는 OQ 프로토콜 개정 |
| 외부 참조 | `pq_item` | 요구사항을 검증하는 PQ 프로토콜 개정 |

### 핵심 이해 포인트

- **논리키나 표시 번호 대신 정확한 개정 PK를 연결합니다.** 원본 항목이 개정되어도 과거 승인 근거의 연결 대상을 새 개정으로 덮어쓰지 않습니다.
- **양 끝의 대상 유형과 관계 의미를 함께 읽습니다.** `IMPLEMENTED_BY`, `ASSESSED_BY`, `VERIFIED_BY`, `MITIGATED_BY`가 관계의 목적을 나타내며 유형에 대응하는 실제 행과 프로젝트 소속을 검증합니다.
- **복수 URS 연결을 지원합니다.** `REQUIREMENT → IQ_ITEM/OQ_ITEM/PQ_ITEM`의 `VERIFIED_BY` 관계를 여러 건 등록해 하나의 시험이 검증하는 요구사항들을 표현합니다.
- **관계의 존재와 수행 완료는 다릅니다.** 시험 항목에 연결되어 있다는 사실만으로 그 시험이 성공·승인되었다고 판단하지 않습니다. 수행 결과는 [5번](#erd-05)·[6번](#erd-06)·[7번](#erd-07) ERD의 기록을 조회합니다.
- **RTM은 대시보드 조회 기능입니다.** 진행률의 독립 수행 단계나 [10번 VSR](#erd-10)의 활동 요약 행으로 추가하지 않습니다.

[↑ 목차로](#top)

---

<a id="erd-10"></a>

## 10. VSR 종합 보고

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/10-vsr-summary)

**주 소속 2개 · 외부 참조 12개 · 총 14개 테이블**

### 관계 구조

```text
vsr_assessment.project_id --FK--> validation_project.project_id
vsr_item.vsr_id --FK--> vsr_assessment.vsr_id
vsr_item.project_activity_id --FK--> project_activity.project_activity_id
project_activity.project_id --FK--> validation_project.project_id
vsr_item.document_revision_id --FK--> deliverable_revision.document_revision_id
vsr_assessment.workflow_instance_id --FK--> workflow_instance.workflow_instance_id
vsr_assessment.approval_signature_id --FK--> electronic_signature.signature_id
vsr_assessment.created_by / updated_by --FK--> app_user.user_id

vsr_item.source_revision_refs ..논리..> vp_plan / qia_module_item / vendor_audit
vsr_item.source_revision_refs ..논리..> iq_execution / oq_execution / pq_execution
deliverable_revision.source_refs ..논리..> vp_plan / qia_module_item / vendor_audit
deliverable_revision.source_refs ..논리..> iq_execution / oq_execution / pq_execution
deliverable_revision.source_refs ..논리..> vsr_assessment
```

### 구조 개요

`vsr_assessment`는 밸리데이션 종합 보고의 개정·결론·업무 확인 상태를 관리하고, `vsr_item`은 프로젝트에서 선택한 수행 활동별 집계 결과를 저장합니다. 각 상세행은 해당 프로젝트 활동, 필요한 경우 생성 산출물 개정, 집계 근거가 된 실제 업무 개정 목록을 연결합니다.

집계 시점의 표시값과 승인정보는 `source_snapshot`에 보존합니다. 원본의 최신 상태를 계속 덮어쓰는 구조가 아니므로, 승인된 VSR은 당시 근거와 결과를 유지하고 이후 원본 변경은 새 VSR 개정에서 재집계합니다. 그림의 VP·QIA·VA·시험 수행은 근거 연결을 설명하는 대표 대상입니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `vsr_assessment` | 종합 결론, VSR 개정, 원본 기준값, 업무 확인·서명 |
| 주 소속 | `vsr_item` | 선택 활동별 승인 상태·일탈 현황·원본 개정·표시값의 집계 사본 |
| 외부 참조 | `validation_project` | 종합 보고의 대상 프로젝트 |
| 외부 참조 | `project_activity` | VSR에 집계하는 프로젝트의 선택 수행 활동 |
| 외부 참조 | `deliverable_revision` | 활동 요약에 연결된 산출물 개정 및 업무 원본 참조 |
| 외부 참조 | `app_user` | VSR 작성자·수정자와 원본 승인 관련 사용자 |
| 외부 참조 | `workflow_instance` | VSR 업무 확인 및 원본·산출물의 승인 절차 |
| 외부 참조 | `electronic_signature` | VSR 최종 확인과 원본 수행·승인에 연결된 서명 |
| 외부 참조 | `vp_plan` | VP 활동 집계에 사용하는 검증 계획 개정 |
| 외부 참조 | `qia_module_item` | QIA 활동 집계에 사용하는 모듈 평가 개정 |
| 외부 참조 | `vendor_audit` | VA 활동 집계에 사용하는 공급업체 감사 개정 |
| 외부 참조 | `iq_execution` | IQ 결과 집계의 정확한 수행 회차·결과 정정 기록 |
| 외부 참조 | `oq_execution` | OQ 결과 집계의 정확한 수행 회차·결과 정정 기록 |
| 외부 참조 | `pq_execution` | PQ 결과 집계의 정확한 수행 회차·결과 정정 기록 |

### 핵심 이해 포인트

- **선택한 실제 수행 활동만 집계합니다.** RTM은 제외하며, VSR 자체 확인 상태는 헤더에 표시합니다. VSR 자신을 상세 활동으로 재귀 집계하지 않습니다.
- **업무 확인과 출력 산출물 승인은 구분합니다.** `vsr_assessment.status`와 산출물 개정의 승인 상태는 별개입니다. 업무 승인만 있고 출력 문서를 아직 생성하지 않았다면 상세행의 `document_revision_id`는 비어 있을 수 있습니다.
- **정확한 원본 개정과 표시 사본을 함께 보존합니다.** `source_revision_refs`는 `{table_name, record_id, version}` 배열이고 `source_snapshot`은 집계 당시 표·승인정보입니다. 참조 대상은 같은 프로젝트의 실제 개정이어야 합니다.
- **변경 감지와 재확인을 분리합니다.** `source_fingerprint`로 근거 변경을 식별하고, 승인 완료된 VSR의 기존 상세행을 덮어쓰는 대신 새 개정을 작성해 다시 확인받습니다.
- **그림의 근거 대상은 전체 허용 유형의 일부입니다.** URS·FDS·DDS·DQ·FRA도 [4번 ERD](#erd-04)의 정확한 개정을 같은 방식으로 연결합니다. 시험 결과의 근거는 시험 항목 PK가 아닌 수행 PK로 식별하며, 일탈 표시값은 해당 활동의 일탈에서 집계합니다.

[↑ 목차로](#top)

---

<a id="erd-11"></a>

## 11. 결재·전자서명·감사기록

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/11-workflow-signature-audit)

**주 소속 7개 · 외부 참조 8개 · 총 15개 테이블**

### 관계 구조

```text
workflow_instance --FK--> validation_project
workflow_instance --FK--> project_workflow_config
workflow_step --FK--> workflow_instance
workflow_step_assignee --FK--> workflow_step
workflow_step_assignee --FK--> app_user (주 담당자·대체 담당자)
approval_action --FK--> workflow_step
approval_action --FK--> workflow_step_assignee
approval_action --FK--> electronic_signature --FK--> app_user

project_workflow_config --FK--> validation_project
project_workflow_config --FK--> project_activity
project_workflow_config --FK--> electronic_signature (설정 적용 서명)
project_workflow_config.route_definition ..논리..> app_user
audit_trail --FK--> app_user (사용자 행위의 수행자)

requirement --FK--> workflow_instance (업무 승인·폐기 승인)
deliverable_revision --FK--> workflow_instance
deviation_action_round --FK--> workflow_instance
deviation_action_round --FK--> electronic_signature
project_closure_request --FK--> workflow_instance

workflow_instance / electronic_signature / audit_trail
  ..논리..> requirement / deliverable_revision / deviation_action_round
  ..논리..> system_asset / project_closure_request
electronic_signature / audit_trail ..논리..> project_workflow_config
```

### 구조 개요

업무별 승인 대상에 공통 결재 절차를 연결하는 구조입니다. `workflow_instance`가 대상의 테이블·PK·버전과 승인 구분을 식별하고, `workflow_step`과 `workflow_step_assignee`가 실제 처리 순서와 담당자를 관리합니다. `approval_action`은 누가 어떤 배정으로 처리했는지와 그때의 전자서명을 연결합니다.

프로젝트 결재선 설정은 재사용할 경로이며, 개별 Workflow는 실제 상신 건입니다. `electronic_signature`는 서명 원문과 해시를 보존하고, `audit_trail`은 변경·접속·내보내기 등 행위 이력을 기록합니다. 요구사항, 산출물 개정, 일탈 조치 회차, 인벤토리, 종료 요청은 서로 다른 승인 대상을 보여주는 대표 외부 참조입니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `workflow_instance` | 대상의 정확한 버전과 승인 구분에 연결되는 결재 실행 건 |
| 주 소속 | `workflow_step` | 검토·승인 단계의 순서, 직렬·병렬 처리 방식, 진행 상태 |
| 주 소속 | `workflow_step_assignee` | 단계별 주 담당자·대체 담당자와 배정별 처리 상태 |
| 주 소속 | `approval_action` | 상신·검토·승인·반려·취소의 실제 처리자, 의견, 서명 연결 |
| 주 소속 | `project_workflow_config` | 프로젝트 기본 또는 활동별 결재선의 버전과 적용 이력 |
| 주 소속 | `electronic_signature` | 서명자, 대상 버전, 재인증 결과, 서명 원문과 해시 |
| 주 소속 | `audit_trail` | 변경 전후 값, 사유, 수행자, 요청·세션 단위 감사기록 |
| 외부 참조 | `app_user` | 상신자·담당자·처리자·서명자·감사 행위자 |
| 외부 참조 | `validation_project` | 프로젝트 결재와 결재선 설정의 소속 범위 |
| 외부 참조 | `project_activity` | 활동별 결재선 설정을 적용하는 수행 활동 |
| 외부 참조 | `requirement` | 업무 항목 개정 승인과 폐기 승인 대상의 예시 |
| 외부 참조 | `deliverable_revision` | 업무 항목 승인과 별개로 승인하는 산출물 문서 개정 |
| 외부 참조 | `deviation_action_round` | 일탈의 특정 조치 회차·개정에 대한 승인과 서명 대상 |
| 외부 참조 | `system_asset` | 시스템 PK와 개정번호로 식별하는 인벤토리 승인 대상 |
| 외부 참조 | `project_closure_request` | 요청 버전별 프로젝트 종료 결재 대상 |

### 핵심 이해 포인트

- **경로 설정과 실행 이력은 다릅니다.** `project_workflow_config`의 설정을 적용하는 행위는 `CONFIG_APPLY` 전자서명으로 기록합니다. 개별 업무의 검토·승인은 `workflow_instance`와 단계·담당 배정으로 추적합니다.
- **업무 대상의 연결은 두 방향으로 표현됩니다.** 업무 행의 `workflow_instance_id` 등은 FK이지만, Workflow와 서명의 `target_table_name + target_record_id + target_version`은 다형 참조입니다. 이 논리 참조가 모든 대상 테이블에 대한 물리 FK를 뜻하지는 않습니다.
- **처리 이력은 실제 담당 배정과 일치해야 합니다.** 검토·승인·반려는 담당 배정당 유효한 처리 한 건을 남기며, 처리 단계·처리자·서명 대상이 일치해야 합니다. 상신·취소에는 담당 배정 ID를 두지 않고, 취소는 서명 없이 사유와 감사기록을 남깁니다.
- **전자서명은 원문을 함께 보존합니다.** `content_hash`는 형식 버전을 포함한 `signed_payload`의 정규화 JSON에 대한 해시입니다. 서명 이후의 본문 변경은 새 개정·새 서명으로 처리합니다.
- **감사기록과 결재기록은 역할이 다릅니다.** 감사기록은 추가 전용 행위 이력이며, 동일 요청의 여러 변경을 `request_id`로 묶습니다. 승인 여부와 담당 배정의 처리는 Workflow 이력에서 확인합니다.

[↑ 목차로](#top)

---

<a id="erd-12"></a>

## 12. 산출물·문서 개정·파일

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/12-documents-files)

**주 소속 6개 · 외부 참조 9개 · 총 15개 테이블**

### 관계 구조

```text
deliverable_document --FK--> validation_project
deliverable_document --FK--> project_activity --FK--> validation_project
deliverable_revision --FK--> deliverable_document
deliverable_section --FK--> deliverable_revision
deliverable_revision --FK--> workflow_instance
deliverable_revision --FK--> project_system_baseline
deliverable_revision --FK--> file_asset (보관한 PDF)

report_generation --FK--> validation_project
report_generation --FK--> deliverable_revision (산출물 내보내기)
report_generation --FK--> file_asset (보관한 결과 파일)
file_asset --FK--> app_user (업로드자)
evidence_link --FK--> file_asset
evidence_link --FK--> validation_project
evidence_link ..논리..> iq_item / iq_execution / iq_step_execution
evidence_link ..논리..> deliverable_revision

deliverable_revision.source_refs ..논리..> iq_assessment / iq_item / iq_execution
iq_assessment.item_revision_refs ..논리..> iq_item
iq_item --FK--> iq_assessment (최초 등록 상위)
iq_execution --FK--> iq_item
iq_execution --FK--> project_system_baseline
iq_step_execution --FK--> iq_execution
```

### 구조 개요

산출물은 문서 식별, 개정, 목차·본문을 나누어 관리합니다. `deliverable_document`가 프로젝트 활동에 속한 문서 자체를 식별하고, `deliverable_revision`이 개정별 제목·본문 구성·근거 업무·승인 상태를 보존합니다. `deliverable_section`은 해당 개정의 목차와 본문입니다. 문서 생성 당시의 업무 개정과 표는 문서 개정에 고정하므로, 이후 업무 원본이 바뀌어도 과거 문서의 근거를 확인할 수 있습니다.

파일은 `file_asset`에 등록하고, 업무 증적은 `evidence_link`로 해당 업무 기록에 연결합니다. `report_generation`은 산출물 또는 감사기록의 PDF 내보내기 작업을 관리합니다. 이 ERD에서는 IQ를 업무 원본과 증적 연결의 예시로 사용하며, 외부 참조의 전체 시험 구조는 [IQ ERD](#erd-05)에서 확인합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `deliverable_document` | 프로젝트 활동별 산출물의 식별자·문서 구분·문서번호 |
| 주 소속 | `deliverable_revision` | 문서 개정별 근거 목록·표 사본·승인·검증 대상 기준 |
| 주 소속 | `deliverable_section` | 특정 문서 개정에 속한 목차, 본문, 순서, 작성 출처 |
| 주 소속 | `report_generation` | PDF 생성 요청, 조회 조건, 진행 상태, 결과 파일·오류 |
| 주 소속 | `file_asset` | 파일 경로, 원본 바이트 해시, 저장소 버전 등 파일 식별 정보 |
| 주 소속 | `evidence_link` | 프로젝트 업무 기록과 증적 파일 사이의 유형별 연결 |
| 외부 참조 | `validation_project` | 문서·내보내기·증적의 프로젝트 범위 |
| 외부 참조 | `project_activity` | 산출물이 귀속되는 실제 수행 활동 |
| 외부 참조 | `project_system_baseline` | 문서 상신 또는 시험 수행 당시 채택한 검증 대상 기준 |
| 외부 참조 | `app_user` | 작성자·수정자·최종 승인자·요청자·업로드자 |
| 외부 참조 | `workflow_instance` | 문서 개정과 시험 프로토콜·결과의 결재 연결 |
| 외부 참조 | `iq_assessment` | 문서 근거로 사용하는 IQ 평가와 개정별 시험 구성 |
| 외부 참조 | `iq_item` | 근거 또는 증적 대상이 되는 특정 IQ 프로토콜 개정 |
| 외부 참조 | `iq_execution` | 근거 또는 증적 대상이 되는 특정 수행 회차·결과 정정 |
| 외부 참조 | `iq_step_execution` | 개별 시험 절차의 수행 증적을 연결하는 대상 |

### 핵심 이해 포인트

- **문서 개정과 업무 개정은 별도로 관리합니다.** `source_refs`는 문서를 구성한 정확한 업무 개정·수행 회차를 가리킵니다. 문서만 개정할 때 변경 없는 업무 원본은 재사용할 수 있습니다.
- **문서 승인과 원본 업무 승인은 별개입니다.** 문서의 `workflow_instance_id`는 문서 자체의 결재이며, 근거 항목·수행의 승인을 대신하지 않습니다. 승인된 본문·표·근거 연결은 직접 고치지 않고 새 문서 개정을 작성합니다.
- **당시의 기준과 현재 유효성을 구분합니다.** 문서와 수행의 `system_baseline_id`는 각각 해당 시점의 기준을 보존합니다. 원본 변경으로 재승인이 필요해도 과거 승인본을 삭제하지 않고 `invalidated_at`과 사유로 현재 유효성을 표시합니다.
- **파일 해시와 서명 해시는 계산 대상이 다릅니다.** `file_asset.content_hash`는 파일 원본 바이트를 식별합니다. 저장소 버전이 없으면 해당 경로를 덮어쓰지 않으며, 승인된 대상의 첨부도 원본을 보존합니다. 절차별 증적은 절차 수행에, 회차 전체 증적은 수행 행에 연결합니다.
- **내보내기 작업과 보관 파일은 분리됩니다.** 산출물 내보내기는 문서 개정을, 감사 내보내기는 기간·필터를 고정합니다. 생성 결과를 파일 자산으로 보관할 때 `result_file_id`를 연결하며 다운로드만 수행한 경우에는 NULL을 허용합니다.

[↑ 목차로](#top)

---

<a id="erd-13"></a>

## 13. 라이브러리·규정 근거

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/13-library-regulation)

**주 소속 5개 · 외부 참조 7개 · 총 12개 테이블**

### 관계 구조

```text
regulatory_clause --FK--> regulatory_source --FK--> file_asset
regulatory_source --FK--> app_user (문서판 확인자)
file_asset --FK--> app_user (업로드자)

library_item_regulation --FK--> library_item
library_item_regulation --FK--> regulatory_clause
requirement_regulation --FK--> requirement
requirement_regulation --FK--> regulatory_clause

requirement --FK--> library_item
fra_item --FK--> library_item
iq_item --FK--> library_item
oq_item --FK--> library_item
pq_item --FK--> library_item
fra_item --FK--> requirement
fra_item --FK--> regulatory_clause (SOP 근거)
```

### 구조 개요

라이브러리는 URS·FRA·IQ·OQ·PQ에서 재사용할 항목을 제공하고, 규정 근거는 문서판과 조항을 구분해 관리합니다. `library_item`은 모듈별 초안 템플릿이며, 프로젝트의 업무 항목은 `source_library_id`로 출처를 남깁니다. `regulatory_source`가 규정·가이드라인·내부 SOP의 특정 판본을 식별하고, `regulatory_clause`는 그 문서판에 속한 개별 조항입니다.

규정 조항과 라이브러리·요구사항은 각각 연결 테이블을 통해 다대다로 연결합니다. 각 연결에는 적용 근거와 당시 인용 문구가 저장됩니다. FRA의 SOP 연결은 위험 항목이 조항을 직접 참조하는 구조이며, 라이브러리의 일반 근거 연결과 구분합니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `library_item` | URS·FRA·IQ·OQ·PQ에서 가져올 재사용 항목과 모듈별 입력값 |
| 주 소속 | `regulatory_source` | 규정·가이드라인·내부 SOP의 판본, 출처, 검토 상태 |
| 주 소속 | `regulatory_clause` | 특정 문서판에 속한 조항 코드·요약·원문 위치 |
| 주 소속 | `requirement_regulation` | 요구사항 개정과 규정 조항의 연결 및 적용 당시 인용 문구 |
| 주 소속 | `library_item_regulation` | 라이브러리 항목과 규정 조항의 연결 및 적용 근거 |
| 외부 참조 | `file_asset` | 규정 문서판의 원본 첨부와 파일 내용 식별 |
| 외부 참조 | `app_user` | 규정 문서 확인자, 근거 작성·수정자, 파일 업로드자 |
| 외부 참조 | `requirement` | 라이브러리로부터 등록하고 규정 근거를 연결하는 실제 URS 개정 |
| 외부 참조 | `fra_item` | 라이브러리 출처와 URS·SOP 근거를 연결하는 위험 항목 |
| 외부 참조 | `iq_item` | 라이브러리에서 시험 내용을 가져오는 IQ 프로토콜 개정 |
| 외부 참조 | `oq_item` | 라이브러리에서 시험 내용을 가져오는 OQ 프로토콜 개정 |
| 외부 참조 | `pq_item` | 라이브러리에서 시험 내용을 가져오는 PQ 프로토콜 개정 |

### 핵심 이해 포인트

- **라이브러리를 적용한 업무 항목은 독립적으로 관리합니다.** 시험 내용은 등록할 때 복사하며 이후 라이브러리 변경을 자동 반영하지 않습니다. URS를 가져올 때에는 규정 근거 관계도 함께 복사합니다.
- **규정의 판본을 구분합니다.** 문서 코드는 판본과 함께 유일하게 식별하고, 새 판본은 새 행으로 등록합니다. 승인된 업무가 사용한 문서판과 조항은 새 내용으로 덮어쓰지 않습니다.
- **근거는 여러 조항을 연결할 수 있습니다.** `requirement_regulation`과 `library_item_regulation`은 동일 항목·조항의 중복을 방지하면서 여러 근거의 순서와 적용 설명을 보존합니다. 인용 문구는 적용 당시 문서명·판본·조항을 표시하는 사본입니다.
- **새로운 근거 선택과 과거 참조 보존은 구분합니다.** 신규 선택에는 확인된 문서판의 활성 조항을 사용합니다. 조항이 이후 비활성화되어도 이미 승인에 사용된 참조는 유지합니다.
- **이 그림의 연결은 물리 FK입니다.** FRA의 `sop_clause_id`도 조항을 직접 참조합니다. 문서 유형이 내부 SOP인지, 라이브러리 모듈별 필수값이 충족되는지 등의 업무 조건은 별도로 검증합니다.

[↑ 목차로](#top)

---

<a id="erd-14"></a>

## 14. AI 생성·적용

[ERD 열기](https://drawsql.app/teams/minho-kim/diagrams/14-ai-generation)

**주 소속 3개 · 외부 참조 9개 · 총 12개 테이블**

### 관계 구조

```text
ai_generation_job --FK--> validation_project
ai_generation_job --FK--> app_user (요청자)
ai_generation_result --FK--> ai_generation_job
ai_generation_result --FK--> app_user (선택 사용자)
ai_result_item --FK--> ai_generation_result

ai_generation_job.target_entity_id
  ..논리..> requirement / fra_item / iq_item / oq_item / pq_item
  ..논리..> deliverable_revision (문서 생성 요청 대상)

ai_result_item.target_entity_id
  ..논리..> requirement / fra_item / iq_item / oq_item / pq_item
  ..논리..> deliverable_section (문서 본문 적용 대상)

deliverable_section --FK--> deliverable_revision
deliverable_revision.source_refs
  ..논리..> requirement / fra_item / iq_item / oq_item / pq_item
requirement --FK--> validation_project
fra_item --FK--> requirement
```

### 구조 개요

AI 생성 요청, 생성 결과 묶음, 개별 결과 항목을 세 단계로 구분합니다. `ai_generation_job`은 입력 조건·대상·진행 상태를 기록하고, `ai_generation_result`는 사용자 선택과 실제 반영 여부를 관리합니다. `ai_result_item`은 생성된 요구사항·위험 시나리오·시험·문서 본문 초안을 저장하고, 적용 후 실제로 생성하거나 수정한 업무 행을 연결합니다.

기존 업무 항목을 대상으로 생성할 때와 새로운 항목을 미리보기로 생성할 때를 모두 지원합니다. 문서 생성은 특정 `deliverable_revision`을 대상으로 요청하고, 목차별 결과를 적용하면 각각의 `deliverable_section`에 연결합니다. AI 생성과 적용은 업무 초안 작성 과정이며, 그 자체로 승인 완료를 의미하지 않습니다.

### 테이블별 역할 요약

| 구분 | 테이블 | 역할 |
|---|---|---|
| 주 소속 | `ai_generation_job` | 항목·문서 생성 요청의 입력값, 대상, 모델 정보, 진행 상태·오류 |
| 주 소속 | `ai_generation_result` | 생성 결과 묶음의 선택 사용자·선택 시점·반영 여부 |
| 주 소속 | `ai_result_item` | 개별 초안 내용, 표시 순서, 채택 여부, 실제 적용 대상·시각 |
| 외부 참조 | `validation_project` | AI 작업과 적용 업무의 프로젝트 범위 |
| 외부 참조 | `app_user` | 요청자·선택 사용자·생성 및 수정 사용자 |
| 외부 참조 | `requirement` | AI 생성 또는 적용 대상인 사용자 요구사항 개정 |
| 외부 참조 | `fra_item` | AI 위험 시나리오를 적용하는 FRA 항목 개정 |
| 외부 참조 | `iq_item` | AI 시험 초안을 적용하는 IQ 프로토콜 개정 |
| 외부 참조 | `oq_item` | AI 시험 초안을 적용하는 OQ 프로토콜 개정 |
| 외부 참조 | `pq_item` | AI 시험 초안을 적용하는 PQ 프로토콜 개정 |
| 외부 참조 | `deliverable_revision` | 문서 생성 작업이 대상으로 삼는 산출물 개정 |
| 외부 참조 | `deliverable_section` | 생성한 목차별 본문을 실제로 적용하는 문서 섹션 |

### 핵심 이해 포인트

- **생성·선택·적용은 서로 다른 상태입니다.** 작업이 `COMPLETED`여도 사용자가 채택하지 않거나 실제 업무에 적용하지 않을 수 있습니다. 결과 묶음과 개별 항목의 선택·적용 여부를 구분해 확인합니다.
- **새 항목은 적용 전 실제 대상 PK가 없을 수 있습니다.** 신규 항목 생성 작업의 `target_entity_id`는 NULL을 허용합니다. 적용한 결과 항목에는 대상 유형·정확한 PK·적용 시각을 남겨야 합니다.
- **문서 생성 대상과 반영 대상은 다릅니다.** 작업은 `DELIVERABLE_REVISION`, 목차별 적용 결과는 `DELIVERABLE_SECTION`을 참조합니다. 두 대상을 동일한 종류로 해석하지 않습니다.
- **AI 대상 참조는 다형 참조입니다.** `target_entity_type`과 `target_entity_id`를 함께 해석하며, 대상의 존재·소속·개정은 서버에서 확인합니다. 작업·결과·결과 항목의 상하위 연결은 일반 FK로 표현합니다.
- **승인된 업무 원문을 덮어쓰지 않습니다.** AI 결과는 편집 가능한 초안 또는 새 개정에 적용하고, 이후 해당 업무의 일반 편집·검토·승인 절차를 따릅니다.

[↑ 목차로](#top)
