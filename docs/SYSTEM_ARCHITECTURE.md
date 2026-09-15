# DVT Validation Management Platform 데이터 구조 및 ERD 설명 자료

> 기준 문서: `DVT_데이터_테이블_정의서_v0.3.xlsx`  
> 분석 대상: `02_컬럼정의`, `ERD` 시트  
> 목적: DVT 플랫폼의 데이터 구조와 16개 ERD 토픽별 핵심 기능 및 관계를 설명하기 위한 자료

---

## 목차

### 문서 개요

- [시스템 개요](#1-시스템-개요)
- [데이터 구조의 핵심 특징](#2-데이터-구조의-핵심-특징)

### ERD 토픽

1. [조직·사용자·전역 권한 관리](#31-no1-조직사용자전역-권한-관리)
2. [시스템·프로젝트·참여자 관리](#32-no2-시스템프로젝트참여자-관리)
3. [Validation 활동·선후행 조건 관리](#33-no3-validation-활동선후행-조건-관리)
4. [라이브러리·QIA·공급업체 감사 관리](#34-no4-라이브러리qia공급업체-감사-관리)
5. [URS·FDS 관리](#35-no5-ursfds-관리)
6. [DDS·DQ 관리](#36-no6-ddsdq-관리)
7. [FRA 위험평가 관리](#37-no7-fra-위험평가-관리)
8. [IQ·OQ·PQ 적격성 시험 관리](#38-no8-iqoqpq-적격성-시험-관리)
9. [설계·위험 산출물 추적성 관리](#39-no9-설계위험-산출물-추적성-관리)
10. [적격성 시험 추적성·RTM 관리](#310-no10-적격성-시험-추적성rtm-관리)
11. [VSR·일탈 관리](#311-no11-vsr일탈-관리)
12. [Workflow·승인·전자서명 관리](#312-no12-workflow승인전자서명-관리)
13. [파일·증적·파일 정리 관리](#313-no13-파일증적파일-정리-관리)
14. [리포트·알림·백업 운영 관리](#314-no14-리포트알림백업-운영-관리)
15. [AI 생성 작업·결과 관리](#315-no15-ai-생성-작업결과-관리)
16. [Audit Trail 관리](#316-no16-audit-trail-관리)

### 종합 설명

- [전체 발표 연결 순서](#4-전체-발표-연결-순서)
- [전체 소개 문구](#5-전체-소개-문구)
- [공통 핵심 메시지](#6-공통-핵심-메시지)
- [주요 용어 구분](#7-주요-용어-구분)

---

# 1. 시스템 개요

## 1.1 어떤 시스템인가

이 데이터 모델은 **GxP 및 CSV 규제 환경에서 시스템과 장비의 밸리데이션 전 과정을 관리하는 Validation Management Platform**이다.

핵심 업무 범위는 다음과 같다.

1. 밸리데이션 대상 시스템·장비 식별
2. Validation 프로젝트 생성 및 참여자 구성
3. 프로젝트별 Validation 활동 및 선후행 조건 관리
4. QIA 및 공급업체 감사 수행
5. URS, FDS, DDS, DQ, FRA 작성
6. IQ, OQ, PQ 적격성 시험 수행
7. 요구사항과 설계·위험·시험 산출물 간 추적성 관리
8. RTM 및 VSR 생성
9. 문서 검토·승인 및 전자서명
10. 증적 파일 및 일탈 관리
11. Audit Trail 및 운영 이력 관리
12. AI 기반 산출물 초안 생성

따라서 이 시스템은 단순한 문서 저장소가 아니다. **밸리데이션 활동의 순서, 산출물 간 관계, 승인 및 전자서명, 시험 증적, 일탈, 추적성을 구조화하여 관리하는 규제 준수형 업무 플랫폼**이다.

## 1.2 전체 데이터 흐름

```text
조직 및 사용자 구성
    ↓
시스템·장비 식별
    ↓
Validation 프로젝트 생성
    ↓
프로젝트 참여자 및 수행 활동 구성
    ↓
QIA·공급업체 감사
    ↓
URS → FDS → DDS → DQ
    ↓
FRA 위험평가
    ↓
IQ → OQ → PQ 적격성 시험
    ↓
추적성 확인 및 RTM 생성
    ↓
일탈 검토 및 VSR 작성
    ↓
프로젝트 종료
```

Workflow, 전자서명, 증적 파일, Audit Trail은 위 업무 전반에 걸쳐 적용되는 공통 통제 기능이다.

---

# 2. 데이터 구조의 핵심 특징

## 2.1 프로젝트 중심 구조

대부분의 업무 데이터는 `validation_project`를 중심으로 연결된다.

```text
organization
  └─ system_asset
       └─ validation_project
            ├─ project_member
            ├─ project_activity
            ├─ QIA / Vendor Audit
            ├─ URS / FDS / DDS / DQ
            ├─ FRA
            ├─ IQ / OQ / PQ
            ├─ RTM / VSR
            ├─ Workflow / Evidence / Deviation
            └─ AI Generation / Report / Notification
```

하나의 조직은 여러 시스템·장비를 관리할 수 있고, 하나의 시스템·장비에 대해 신규 검증, 변경 검증, 재검증 등 여러 Validation 프로젝트를 생성할 수 있다.

## 2.2 문서 헤더와 상세 항목 분리

주요 밸리데이션 산출물은 문서 단위 테이블과 상세 항목 테이블로 분리된다.

```text
fds_spec       → fds_item / fds_interface
dds_spec       → dds_item
dq_assessment  → dq_item
fra_assessment → fra_item
iq_assessment  → iq_item
oq_assessment  → oq_item
pq_assessment  → pq_item
rtm_assessment → rtm_item
vsr_assessment → vsr_item
```

- 헤더 테이블은 문서 번호, 제목, 버전, 개정 순번, 상태를 관리한다.
- 상세 테이블은 실제 요구사항, 설계, 위험, 시험 및 요약 항목을 관리한다.

## 2.3 버전 및 개정 관리

주요 산출물에는 다음과 같은 개정 관리 정보가 포함된다.

- `version`: 사용자에게 표시되는 문서 버전
- `revision_number`: 시스템에서 관리하는 개정 순번
- `revision_reason`: 신규 작성 또는 개정 사유
- `is_current_version`: 현재 유효한 최신 버전 여부
- `status`, `protocol_status`, `record_status`: 문서 또는 수행 기록의 상태

이를 통해 승인된 기록을 직접 덮어쓰기보다 새로운 Revision을 생성하고 이전 기록을 보존할 수 있다.

## 2.4 다형 참조 구조

다음 공통 테이블은 특정 업무 테이블에 고정된 FK를 두지 않고 대상 유형과 대상 ID를 함께 저장한다.

- `workflow_instance`
- `electronic_signature`
- `audit_trail`
- `traceability_link`
- `evidence_link`
- `deviation`
- AI 관련 테이블

예시는 다음과 같다.

```text
target_entity_type = IQ_ITEM
target_entity_id   = 특정 iq_item의 PK
```

이 구조는 여러 종류의 업무 엔터티에 공통 기능을 적용하기 쉽다. 다만 물리 FK를 설정할 수 없는 관계는 애플리케이션 또는 서비스 계층에서 유효성을 검증해야 한다.

## 2.5 GxP 공통 통제 구조

전체 데이터 모델에는 다음 통제 개념이 적용된다.

- 작성자·수정자와 생성·수정 시각
- 소프트 삭제
- 문서 버전 및 개정 사유
- Workflow 기반 검토·승인
- 전자서명 및 재인증 결과
- 변경 전후 값 Audit Trail
- 증적 파일 연결
- 일탈 조사·해결·승인 이력
- 승인 시점의 RTM 및 VSR 결과 보존

핵심 목적은 업무 데이터뿐 아니라 **누가, 언제, 무엇을, 어떤 이유로 작성·변경·검토·승인했는지**를 추적할 수 있도록 하는 것이다.

---

# 3. 16개 ERD 토픽별 설명

## 3.1 No.1 조직·사용자·전역 권한 관리

**영문명:** Organization, User, and Global Role Management

### 사용 테이블

- `organization`
- `app_user`
- `role`
- `user_role`
- `user_group`
- `user_group_member`
- `group_role`

### 이 ERD가 설명하는 것

플랫폼을 사용하는 조직과 사용자, 역할, 사용자 그룹을 구성하고 **전역 또는 프로젝트 범위의 권한을 부여하는 기본 보안 구조**이다.

### 중심 관계

```text
organization
  └─ app_user
       ├─ user_role ─ role
       └─ user_group_member ─ user_group ─ group_role ─ role
```

### 테이블별 기능

#### `organization`

- 고객사, 본사, 공장, 연구소 등 플랫폼의 최상위 관리 단위이다.
- 사용자와 시스템·장비가 조직에 소속된다.

#### `app_user`

- 로그인 계정, 사용자 이름, 이메일, 부서, 직급, 계정 상태를 관리한다.
- 작성자, 검토자, 승인자, 시험 수행자 등 모든 사용자 참조의 기준이 된다.

#### `role`

- 시스템 관리자, 작성자, 검토자, 승인자 등 권한 역할의 마스터이다.

#### `user_role`

- 특정 사용자에게 역할을 직접 부여한다.
- 사용자와 역할 간 N:M 관계를 해소한다.

#### `user_group`

- QA 검토자 그룹과 같은 조직 단위 사용자 그룹을 정의한다.

#### `user_group_member`

- 사용자가 어떤 그룹에 소속되는지 관리한다.
- 참여 시작·종료 시점과 구성원 상태도 기록한다.

#### `group_role`

- 사용자 그룹에 역할을 일괄 부여한다.
- `GLOBAL`과 `PROJECT` 범위를 구분하여 권한을 적용한다.

### 핵심 설명

> 사용자에게 역할을 직접 부여할 수도 있고 사용자 그룹을 통해 여러 사용자에게 동일 역할을 일괄 부여할 수도 있습니다. 그룹 역할은 전역 권한과 프로젝트별 권한을 구분할 수 있도록 설계되어 있습니다.

### 주요 관계

- 조직 1개에 여러 사용자 존재
- 사용자와 역할은 N:M 관계
- 사용자 그룹과 사용자는 N:M 관계
- 사용자 그룹과 역할은 N:M 관계
- 프로젝트 범위의 그룹 역할은 `validation_project`와 연결

---

## 3.2 No.2 시스템·프로젝트·참여자 관리

**영문명:** System, Project, and Participant Management

### 사용 테이블

- `organization`
- `system_asset`
- `validation_project`
- `project_member`
- `app_user`
- `role`

### 이 ERD가 설명하는 것

어떤 조직의 어떤 시스템·장비를 대상으로 Validation 프로젝트가 수행되며, 누가 어떤 역할로 프로젝트에 참여하는지를 관리한다.

### 중심 관계

```text
organization
  └─ system_asset
       └─ validation_project
            └─ project_member
                 ├─ app_user
                 └─ role
```

### 테이블별 기능

#### `organization`

- 시스템·장비와 사용자가 소속되는 최상위 관리 범위이다.

#### `system_asset`

- 밸리데이션 대상 시스템·장비의 기준정보를 관리한다.
- 관리번호, 유형, 위치, 공급업체, 버전, GAMP 범주, GxP 구분을 저장한다.
- 시스템 식별 상태와 Computerized System 포함 여부도 관리한다.

#### `validation_project`

- 실제 밸리데이션 수행 단위이다.
- 프로젝트 유형, 밸리데이션 레벨, 상태, 시작일, 진행률을 관리한다.
- 정상종료와 강제종료를 구분하고, 강제종료 시 사유와 요청자를 보존한다.

#### `project_member`

- 특정 프로젝트에 참여하는 사용자와 프로젝트 내 역할을 연결한다.
- 프로젝트 참여 상태와 참여 기간을 관리한다.

#### `app_user`

- 프로젝트에 참여하는 실제 사용자이다.

#### `role`

- 프로젝트 참여자가 수행하는 작성자, 검토자, 승인자 등의 역할이다.

### 핵심 설명

> `app_user`와 `role`은 시스템 전체의 사용자와 역할 기준정보이고, `project_member`는 그중 어떤 사용자가 특정 프로젝트에서 어떤 역할을 수행하는지를 정의합니다. 따라서 전역 권한과 프로젝트 참여 권한이 구분됩니다.

### 주요 관계

- 조직 1개에 여러 시스템·장비 존재
- 시스템·장비 1개에 여러 Validation 프로젝트 생성 가능
- 프로젝트 1개에 여러 참여자 존재
- 동일 사용자도 프로젝트별로 다른 역할 수행 가능

---

## 3.3 No.3 Validation 활동·선후행 조건 관리

**영문명:** Validation Activity and Dependency Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `validation_activity`
- `project_activity`
- `activity_dependency`

### 이 ERD가 설명하는 것

프로젝트에서 수행할 Validation 활동을 선택하고, 활동별 상태와 선후행 조건에 따라 다음 업무를 활성화하는 구조이다.

### 중심 관계

```text
validation_activity
        ├─ project_activity ─ validation_project
        └─ activity_dependency
             ├─ predecessor_activity_id
             └─ successor_activity_id
```

### 테이블별 기능

#### `validation_activity`

- 시스템 식별, VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR 등 활동의 마스터이다.
- 활동 코드, 명칭, 기본 표시 순서를 관리한다.

#### `project_activity`

- 활동 마스터 중 특정 프로젝트에서 실제 수행할 활동을 선택한다.
- 수행 대상 여부, 필수 여부, 활동 상태를 관리한다.
- 주요 상태는 `LOCKED`, `READY`, `IN_PROGRESS`, `COMPLETED`, `APPROVED`, `SKIPPED`이다.

#### `activity_dependency`

- 후행 활동을 활성화하기 위한 선행 조건을 정의한다.
- 단순 승인 상태뿐 아니라 다음 조건도 표현할 수 있다.
  - 프로젝트 컨텍스트 확정
  - GxP 범위 확정
  - 추적성 존재
  - 고위험 항목 시험 커버
  - 미해결 일탈 0건
  - 선택된 모든 활동 승인 완료

### 핵심 설명

> 이 구조는 단순한 진행률 관리가 아니라 실제 선행 산출물의 승인 상태와 추적성, 위험, 일탈 조건을 기준으로 다음 활동을 활성화하기 위한 구조입니다.

### 주요 관계

- 활동 마스터와 프로젝트는 `project_activity`를 통해 N:M 연결
- `activity_dependency`는 `validation_activity`를 선행 활동과 후행 활동으로 각각 참조
- 프로젝트 진행률은 개별 활동의 원천 상태를 기반으로 계산하는 구조

---

## 3.4 No.4 라이브러리·QIA·공급업체 감사 관리

**영문명:** Library, QIA, and Vendor Audit Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `library_item`
- `qia_assessment`
- `qia_module_item`
- `vendor_audit`

### 이 ERD가 설명하는 것

프로젝트 초기에 표준 항목을 재사용하고, 대상 시스템의 GxP 영향 및 21 CFR Part 11 적용 여부와 공급업체 적합성을 평가한다.

### 테이블별 기능

#### `library_item`

- URS, IQ, OQ에서 재사용할 수 있는 표준 요구사항과 시험 항목을 저장한다.
- 카테고리, 요구사항 또는 절차, 기대 결과, 수용 기준, 근거 규정을 관리한다.
- 프로젝트에 직접 종속되지 않는 기준정보이다.

#### `qia_assessment`

- 프로젝트 단위 QIA 평가 문서 헤더이다.
- 21 CFR Part 11 적용 여부와 GxP 범위 확정 결과를 관리한다.
- 문서 버전, 개정 순번, 최신 버전, 상태를 포함한다.

#### `qia_module_item`

- QIA 내 모듈·프로세스별 Q1부터 Q10까지의 상세 평가를 관리한다.
- 각 항목의 답변과 최종 GxP 또는 Non-GxP 결과를 저장한다.

#### `vendor_audit`

- 공급업체 감사 계획과 수행 결과를 관리한다.
- 감사 방식, 감사일, 감사자, 감사 결과를 기록한다.
- Critical, Major, Minor 결함 건수도 보존한다.

### 핵심 설명

> 라이브러리는 재사용 가능한 표준 콘텐츠이고, QIA와 공급업체 감사는 실제 프로젝트에 종속되는 평가 결과입니다. 특히 QIA 결과는 Validation 범위와 이후 수행 활동을 결정하는 중요한 기준이 됩니다.

### 주요 관계

- 프로젝트 1개에 여러 QIA Revision 존재 가능
- QIA 1건에 여러 모듈·프로세스 평가 항목 존재
- 공급업체 감사는 프로젝트 기준으로 관리
- 라이브러리는 독립 마스터로 관리

---

## 3.5 No.5 URS·FDS 관리

**영문명:** URS and FDS Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `requirement`
- `fds_spec`
- `fds_item`
- `fds_interface`

### 이 ERD가 설명하는 것

사용자의 요구사항을 구조화하고, 해당 요구사항을 시스템 기능·화면·인터페이스 설계로 구체화하는 과정이다.

### 중심 관계

```text
validation_project
  ├─ requirement
  └─ fds_spec
       ├─ fds_item
       └─ fds_interface
```

### 테이블별 기능

#### `requirement`

- 프로젝트별 URS 상세 요구사항을 관리한다.
- 항목 번호, 카테고리, 기능명, 요구사항 상세, 근거 규정을 저장한다.
- 요구사항 항목 자체의 버전, 개정 순번, 최신 여부와 상태를 관리한다.

#### `fds_spec`

- 프로젝트별 FDS 문서 헤더이다.
- 문서 번호, 문서명, 버전, 개정, 상태를 관리한다.

#### `fds_item`

- FDS의 기능 및 화면 설계 상세 항목이다.
- 기능명, 설명, 관련 화면과 개별 상태를 저장한다.

#### `fds_interface`

- 시스템 간 인터페이스 정의를 관리한다.
- 소스 시스템, 대상 시스템, 연동 데이터, 연동 주기, 연동 방식을 기록한다.

### 핵심 설명

> URS가 사용자가 필요로 하는 기능을 정의한다면 FDS는 그 요구사항이 시스템 기능과 화면, 인터페이스로 어떻게 구현되는지를 정의합니다. FDS는 문서 헤더, 상세 기능, 인터페이스 영역으로 분리되어 있습니다.

### 주요 관계

- 프로젝트와 URS는 1:N 관계
- 프로젝트와 FDS 문서는 1:N 관계
- FDS 문서와 기능 항목은 1:N 관계
- FDS 문서와 인터페이스 항목은 1:N 관계
- URS와 FDS 항목 간 공식 추적 관계는 `traceability_link`에서 관리

---

## 3.6 No.6 DDS·DQ 관리

**영문명:** DDS and DQ Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `requirement`
- `fds_spec`
- `fds_item`
- `dds_spec`
- `dds_item`
- `dq_assessment`
- `dq_item`

### 이 ERD가 설명하는 것

FDS를 기반으로 데이터베이스, 컴포넌트, 인터페이스 등의 상세 설계를 정의하고, 해당 설계가 URS를 적절하게 충족하는지 평가한다.

### 테이블별 기능

#### `dds_spec`

- DDS 문서 헤더이다.
- 문서 번호, 버전, 개정 순번, 최신 여부와 승인 상태를 관리한다.

#### `dds_item`

- DDS 상세 설계 항목이다.
- `DATABASE`, `COMPONENT`, `INTERFACE`, `SECURITY`, `BATCH` 등의 설계 유형을 구분한다.

#### `dq_assessment`

- 설계 적격성 평가 문서 헤더이다.
- 문서 번호, 버전, 개정, 최신 여부와 상태를 관리한다.

#### `dq_item`

- URS별로 FDS와 DDS 설계가 적절하게 반영되었는지 평가한다.
- `requirement_id`로 평가 대상 URS와 연결된다.
- 설계 적격성 판정, 검토자, 비고를 관리한다.

#### `requirement`, `fds_spec`, `fds_item`

- DQ 평가 기준이 되는 요구사항과 기능 설계 정보이다.

### 핵심 설명

> DDS는 구현 관점의 상세 설계이고, DQ는 해당 설계가 사용자 요구사항을 충족하는지를 검토하는 평가 영역입니다. 이 ERD는 URS에서 FDS, DDS, DQ로 이어지는 설계 검증 흐름을 보여줍니다.

### 관계 해석 시 유의사항

`dq_item`에는 화면 표시용 FDS·DDS 매핑 값이 포함되어 있다. 하지만 산출물 간 공식적인 범용 추적 관계는 `traceability_link`를 기준으로 설명하는 것이 적절하다.

---

## 3.7 No.7 FRA 위험평가 관리

**영문명:** FRA Risk Assessment Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `requirement`
- `fra_assessment`
- `fra_item`

### 이 ERD가 설명하는 것

URS 또는 시스템 기능별 위험 시나리오를 평가하고, 위험 수준에 따라 시험, SOP, 조치 불필요 등의 완화 전략을 결정한다.

### 중심 관계

```text
validation_project
  ├─ requirement
  └─ fra_assessment
       └─ fra_item
            └─ requirement
```

### 테이블별 기능

#### `fra_assessment`

- 프로젝트별 FRA 문서 헤더이다.
- 문서 번호, 버전, 개정, 최신 여부와 승인 상태를 관리한다.

#### `fra_item`

- 기능별 위험평가 상세 항목이다.
- 위험 시나리오, 제품 영향, 발생가능성, 탐지가능성을 평가한다.
- 위험값과 위험등급을 산출하고 완화 전략을 정의한다.
- 필요하면 관련 검증 시험 참조값을 관리한다.

#### `requirement`

- 위험평가 대상이 되는 URS이다.
- `fra_item.requirement_id`로 연결되며, URS와 직접 연결되지 않는 위험 항목도 허용된다.

### 핵심 설명

> FRA의 목적은 모든 기능을 동일한 강도로 시험하는 것이 아니라 위험 수준을 평가하여 고위험 기능에 적절한 시험과 통제를 집중하는 것입니다.

### 주요 관계

- 프로젝트 1개에 여러 FRA Revision 존재 가능
- FRA 문서 1건에 여러 위험 항목 존재
- 위험 항목은 필요한 경우 URS와 연결
- 위험평가 결과는 이후 IQ·OQ·PQ 시험과 RTM 커버리지 판단에 활용

---

## 3.8 No.8 IQ·OQ·PQ 적격성 시험 관리

**영문명:** IQ, OQ, and PQ Qualification Testing Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `iq_assessment`
- `iq_item`
- `oq_assessment`
- `oq_item`
- `pq_assessment`
- `pq_item`
- `deviation`

### 이 ERD가 설명하는 것

설치, 운전, 성능 적격성 시험의 프로토콜과 수행 결과를 구분하여 관리하고, 시험 중 발생한 일탈을 연결한다.

### 테이블별 기능

#### `iq_assessment`, `oq_assessment`, `pq_assessment`

- 각 적격성 시험의 문서 헤더이다.
- 문서 버전과 개정 정보를 관리한다.
- 프로토콜 승인 상태와 실제 수행 레코드 승인 상태를 구분한다.

#### `iq_item`

- 설치 환경, 하드웨어 및 소프트웨어 구성이 규격에 맞는지 확인한다.

#### `oq_item`

- 기능, 권한, 전자서명, Audit Trail, 백업 등 시스템 기능이 의도대로 동작하는지 확인한다.

#### `pq_item`

- 실제 운영 또는 생산 조건에서 시스템이 일관된 성능을 내는지 확인한다.

#### `deviation`

- 시험 중 기대 결과와 실제 결과가 불일치하는 경우 발생한 일탈을 관리한다.
- IQ·OQ·PQ 항목을 대상 유형과 대상 ID로 연결한다.

### 공통 시험 항목 구조

```text
assessment
  └─ item
       ├─ test_description
       ├─ expected_result
       ├─ actual_result
       ├─ qualification_result
       ├─ executed_by
       └─ executed_at
```

### IQ·OQ·PQ의 차이

- **IQ:** 시스템이 올바르게 설치되었는가
- **OQ:** 기능이 정의된 규격대로 동작하는가
- **PQ:** 실제 운영 조건에서도 지속적으로 성능을 충족하는가

### 핵심 설명

> 각 적격성 시험은 시험 절차인 프로토콜 승인과 실제 수행 결과인 레코드 승인을 구분합니다. 프로토콜이 승인된 후 실제 시험을 수행하고, 실제 결과와 판정 및 증적을 기록합니다.

---

## 3.9 No.9 설계·위험 산출물 추적성 관리

**영문명:** Design and Risk Deliverable Traceability Management

### 사용 테이블

- `app_user`
- `validation_project`
- `requirement`
- `fds_spec`
- `fds_item`
- `dds_spec`
- `dds_item`
- `dq_assessment`
- `dq_item`
- `fra_assessment`
- `fra_item`
- `traceability_link`

### 이 ERD가 설명하는 것

URS, FDS, DDS, DQ, FRA 항목 간 관계를 공통 추적 구조로 연결하여 요구사항이 설계되고 평가되며 위험 통제를 받는 흐름을 관리한다.

### 중심 테이블: `traceability_link`

`traceability_link`는 출발 엔터티와 대상 엔터티를 연결하는 범용 추적 관계 테이블이다.

주요 관계 유형은 다음과 같다.

- `IMPLEMENTED_BY`: 요구사항이 설계 또는 기능으로 구현됨
- `ASSESSED_BY`: 요구사항 또는 설계가 평가 항목으로 검토됨
- `VERIFIED_BY`: 요구사항 또는 설계가 시험으로 검증됨
- `MITIGATED_BY`: 위험이 시험 또는 통제로 완화됨

### 연결 대상

- `requirement`
- `fds_item`
- `dds_item`
- `dq_item`
- `fra_item`

문서 헤더 테이블은 각 상세 항목의 소속 문서, 프로젝트, 버전과 상태를 제공한다.

### 관계 예시

```text
URS-001
  ├─ IMPLEMENTED_BY → FDS-001
  ├─ IMPLEMENTED_BY → DDS-001
  ├─ ASSESSED_BY    → DQ-001
  └─ ASSESSED_BY    → FRA-001
```

### 핵심 설명

> 요구사항과 설계·위험 항목 사이의 실제 추적 관계는 각 업무 테이블에 여러 FK를 추가하는 대신 `traceability_link` 하나로 통합했습니다. 새로운 산출물 유형이 추가되어도 동일한 방식으로 확장할 수 있습니다.

### 구조상 유의사항

다형 참조 구조이므로 `source_entity_id`와 `target_entity_id`에는 대상 업무 테이블로의 물리 FK가 설정되지 않는다. 엔터티 유형별 유효성은 서비스 또는 애플리케이션 계층에서 검증해야 한다.

---

## 3.10 No.10 적격성 시험 추적성·RTM 관리

**영문명:** Qualification Test Traceability and RTM Management

### 사용 테이블

- `app_user`
- `validation_project`
- `requirement`
- `iq_assessment`
- `iq_item`
- `oq_assessment`
- `oq_item`
- `pq_assessment`
- `pq_item`
- `traceability_link`
- `rtm_assessment`
- `rtm_item`

### 이 ERD가 설명하는 것

URS 요구사항이 IQ·OQ·PQ 시험으로 적절히 검증되었는지 추적하고, 승인 시점의 RTM 문서와 항목별 커버리지를 관리한다.

### 테이블별 기능

#### `traceability_link`

- URS와 IQ·OQ·PQ 시험 항목 간의 실제 원천 관계를 관리한다.

#### `rtm_assessment`

- 프로젝트 단위 RTM 문서 헤더이다.
- URS 전체 건수, FRA 연계율, 시험 커버리지, 전체 평균 커버리지를 관리한다.
- 승인 시점의 RTM 집계 결과를 스냅샷으로 보존한다.

#### `rtm_item`

- URS별 FDS, DDS, FRA, IQ, OQ, PQ 연결 결과를 한 행으로 보여주는 상세 스냅샷이다.
- 항목별 커버리지와 매핑 실패 여부를 기록한다.

#### `iq_assessment`, `oq_assessment`, `pq_assessment`

- 각 시험 항목이 속한 문서와 승인 상태를 제공한다.

### 처리 구조

```text
requirement
  ├─ traceability_link → iq_item
  ├─ traceability_link → oq_item
  └─ traceability_link → pq_item

원천 관계 집계
  └─ rtm_assessment
       └─ rtm_item
```

### 핵심 설명

> `traceability_link`가 운영 중인 원천 추적 관계라면 `rtm_assessment`와 `rtm_item`은 특정 시점에 생성되고 승인된 RTM 문서의 스냅샷입니다. RTM 승인 이후 원천 관계가 변경되어도 당시 승인된 RTM 내용을 보존할 수 있습니다.

### 핵심 의미

RTM은 단순한 연결 목록이 아니다. 다음 사항을 확인하는 규제 대응용 추적성 산출물이다.

- 미연결 URS
- 설계 미반영 요구사항
- 위험평가 누락
- 미수행 또는 미연결 시험
- 매핑 실패
- 항목별 및 전체 커버리지

---

## 3.11 No.11 VSR·일탈 관리

**영문명:** VSR and Deviation Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `validation_activity`
- `project_activity`
- `rtm_assessment`
- `deviation`
- `vsr_assessment`
- `vsr_item`

### 이 ERD가 설명하는 것

프로젝트 전체 Validation 활동의 수행 결과와 일탈 상태를 종합하여 최종 결론을 작성하고 프로젝트 종료 판단에 활용한다.

### 테이블별 기능

#### `deviation`

- 문서 또는 시험 항목에서 발생한 일탈을 관리한다.
- 일탈 번호, 심각도, 상태, 조사·해결 내용, 해결자, 승인자를 기록한다.

#### `vsr_assessment`

- 프로젝트의 최종 Validation Summary Report이다.
- 최종 적합성 결론과 조건 사항을 관리한다.

#### `vsr_item`

- URS, FDS, DQ, FRA, IQ, OQ, PQ, RTM 등 활동별 문서 번호, 버전, 수행일, 성공·실패 건수, 일탈과 승인 상태를 요약한다.

#### `validation_activity`, `project_activity`

- 프로젝트에서 어떤 활동이 선택되었고 어느 상태까지 진행되었는지 제공한다.

#### `rtm_assessment`

- 요구사항 추적성과 시험 커버리지 결과를 VSR 결론에 제공한다.

### 최종 판단 흐름

```text
project_activity 승인 상태
        +
RTM 추적성 및 커버리지
        +
deviation 해결·종결 상태
        ↓
VSR 종합 결론
        ↓
프로젝트 정상 종료
```

### 핵심 설명

> VSR은 단순한 별도 보고서가 아니라 각 활동의 승인 여부, 시험 결과, RTM 커버리지, 미해결 일탈을 종합하여 최종 밸리데이션 결론을 관리하는 영역입니다.

### 종료 유형 설명

- 정상종료는 선택된 활동의 승인, 추적성, 일탈 조건을 종합해 판단한다.
- 강제종료는 정상 완료 조건과 별개로 종료 유형, 종료 사유, 요청자와 처리 시점을 보존한다.

---

## 3.12 No.12 Workflow·승인·전자서명 관리

**영문명:** Workflow, Approval, and Electronic Signature Management

### 사용 테이블

- `organization`
- `app_user`
- `role`
- `system_asset`
- `validation_project`
- `project_member`
- `workflow_instance`
- `workflow_step`
- `approval_action`
- `electronic_signature`

### 이 ERD가 설명하는 것

다양한 밸리데이션 문서를 공통 Workflow로 상신하고, 단계별 검토·승인·반려 처리와 전자서명 증적을 남긴다.

### 중심 관계

```text
workflow_instance
  └─ workflow_step
       └─ approval_action
            └─ electronic_signature
```

### 테이블별 기능

#### `workflow_instance`

- 특정 문서 Revision에 대한 Workflow 전체 인스턴스이다.
- 상신자, 대상 문서, 대상 버전, 현재 단계, 전체 Workflow 상태를 관리한다.

#### `workflow_step`

- Workflow의 개별 검토·승인 단계를 관리한다.
- 단계 순서, 단계 유형, 담당자, 처리 기한, 상태를 저장한다.

#### `approval_action`

- 상신, 검토, 승인, 반려, 취소 등의 실제 처리 이력이다.
- 처리자, 처리 의견, 반려 사유, 처리 시각을 기록한다.

#### `electronic_signature`

- 승인·반려 등의 처리에 대한 전자서명 증적이다.
- 서명자, 서명 의미, 서명 시각, 대상 버전을 저장한다.
- 서명 시점의 내용 해시와 재인증 방법 및 결과를 보존한다.

#### `project_member`, `role`

- 해당 프로젝트에서 누가 작성자, 검토자, 승인자로 지정될 수 있는지 판단하는 기준이다.

### 핵심 설명

> Workflow는 업무가 어느 단계에 있는지를 관리하고, `approval_action`은 실제로 누가 어떤 처리를 했는지를 기록하며, `electronic_signature`는 그 처리에 대한 규제 준수형 서명 증적을 보존합니다.

### 개념 구분

- **Workflow 상태:** 현재 업무가 어느 단계에 있는지
- **승인 처리 이력:** 실제 수행된 상신·검토·승인·반려 행위
- **전자서명:** 해당 행위에 대한 본인 확인과 서명 증적
- **Audit Trail:** 처리 과정에서 발생한 데이터 변경 이력

---

## 3.13 No.13 파일·증적·파일 정리 관리

**영문명:** File, Evidence, and File Cleanup Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `file_cleanup_execution`
- `file_asset`
- `evidence_link`

### 이 ERD가 설명하는 것

업로드 파일, 시험 증적, 생성 리포트 등의 저장 메타데이터와 업무 항목 간 연결, 임시·만료파일 정리 이력을 관리한다.

### 중심 관계

```text
file_cleanup_execution
        └─ file_asset
             └─ evidence_link
                  └─ 각 업무 문서 또는 시험 항목
```

### 테이블별 기능

#### `file_asset`

- 실제 파일의 저장 메타데이터를 관리한다.
- 원본 파일명, 저장 경로, MIME 타입, 크기, 파일 종류를 저장한다.
- 첨부파일, 증적, 리포트, 내보내기 파일을 구분한다.

#### `evidence_link`

- 파일과 실제 업무 문서 또는 시험 항목을 연결한다.
- 하나의 파일을 여러 항목에 연결하거나 하나의 항목에 여러 파일을 연결할 수 있다.
- 시험 결과, 화면 캡처, 로그, 리포트, 승인 문서 등 증적 유형을 구분한다.

#### `file_cleanup_execution`

- 임시파일, 만료파일, 고아 파일 정리 작업의 실행 이력을 관리한다.
- 조회 건수, 정리 대상 건수, 성공·실패 건수와 재시도 정보를 기록한다.

### 핵심 설명

> `file_asset`은 파일 자체의 메타데이터이고, `evidence_link`는 그 파일이 어떤 문서나 시험 항목의 증적인지를 표현합니다. 두 영역을 분리했기 때문에 동일 파일을 여러 업무 항목에 연결할 수 있습니다.

### 파일 정리의 의미

파일 정리는 규정상 보존해야 하는 증적을 임의로 삭제하는 기능이 아니다. 보존 대상이 아닌 임시·만료·고아 파일을 통제된 절차로 정리하고 처리 결과를 기록하는 운영 기능이다.

---

## 3.14 No.14 리포트·알림·백업 운영 관리

**영문명:** Report, Notification, and Backup Operations Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `workflow_instance`
- `workflow_step`
- `file_asset`
- `report_schedule`
- `report_generation`
- `notification_delivery`
- `backup_execution`

### 이 ERD가 설명하는 것

플랫폼의 비동기 운영 업무인 리포트 생성, 알림 발송, 백업 실행과 실패·재시도 이력을 관리한다.

### 테이블별 기능

#### `report_schedule`

- 월간 Audit Trail 리포트와 같은 정기 리포트 실행 주기를 정의한다.
- 조회기간, 조회 조건, 출력형식, 다음 실행 시각을 관리한다.

#### `report_generation`

- 실제 리포트 생성 작업이다.
- 사용자 요청 실행과 정기 실행을 구분한다.
- 처리 상태, 조회 조건, 실패 사유와 재시도 정보를 저장한다.
- 생성된 결과 파일은 `file_asset`과 연결한다.

#### `notification_delivery`

- 승인 요청, 검토 요청, 처리 지연, 반려, 완료 등의 알림 발송 이력이다.
- Workflow, 단계, 수신자, 발송 채널, 상태, 실패 및 재시도 정보를 관리한다.

#### `backup_execution`

- 데이터베이스와 파일 저장소의 정기 또는 수동 백업 실행 이력이다.
- 백업 범위, 상태, 저장 위치, 파일 크기, 실패 및 재시도 정보를 관리한다.

### 공통 작업 상태 패턴

```text
PENDING
  → PROCESSING
  → COMPLETED

실패 시
  → RETRY_WAIT
  → 재실행
  → COMPLETED 또는 FAILED
```

### 핵심 설명

> 이 테이블들은 인프라 구성 자체를 표현하는 것이 아니라 운영 과정에서 발생한 작업 요청과 처리 결과, 실패 및 재시도 이력을 데이터로 관리하기 위한 구조입니다.

### 주요 관계

- 리포트 일정 1건에서 여러 리포트 생성 작업 발생 가능
- 리포트 생성 결과는 `file_asset`으로 관리
- Workflow 인스턴스·단계와 알림 발송 이력 연결
- 백업 실행 이력은 독립적인 운영 기록으로 관리

---

## 3.15 No.15 AI 생성 작업·결과 관리

**영문명:** AI Generation Job and Result Management

### 사용 테이블

- `organization`
- `app_user`
- `system_asset`
- `validation_project`
- `ai_generation_job`
- `ai_generation_result`
- `ai_result_item`

### 이 ERD가 설명하는 것

AI를 이용하여 URS, FRA, IQ, OQ, PQ 등의 항목이나 문서 초안을 생성하고, 생성 결과의 선택과 실제 산출물 반영 여부를 관리한다.

### 중심 관계

```text
ai_generation_job
  └─ ai_generation_result
       └─ ai_result_item
```

### 테이블별 기능

#### `ai_generation_job`

- AI 생성 요청 단위이다.
- 프로젝트, 작업 유형, 대상 엔터티, 사용 모델, 입력 파라미터를 기록한다.
- 실행 상태와 실패·재시도 정보를 관리한다.

#### `ai_generation_result`

- 하나의 AI 작업에서 생성된 결과 집합이다.
- 결과 제목, 사용자 선택 여부, 실제 반영 여부를 관리한다.

#### `ai_result_item`

- 결과 집합 안의 개별 생성 항목이다.
- URS 요구사항, FRA 위험 시나리오, IQ·OQ·PQ 시험, 문서 섹션 등의 내용을 저장한다.
- 개별 항목별 선택 여부와 적용 대상도 관리한다.

### 처리 흐름

```text
AI 생성 요청
  → ai_generation_job
       → ai_generation_result
            → ai_result_item
                 → 사용자 검토·선택
                      → 실제 업무 테이블 반영
                           → 공식 검토·승인 Workflow
```

### 핵심 설명

> AI 결과는 생성 즉시 승인된 공식 산출물이 되지 않습니다. 생성 작업과 원본 결과를 별도로 보존하고, 사용자가 결과 또는 개별 항목을 검토하고 선택한 후 실제 업무 테이블에 반영합니다.

### 핵심 통제 포인트

- 어떤 모델과 입력 조건으로 생성했는지 기록
- 생성 원본과 실제 적용 결과 분리
- 사용자 선택 및 반영 여부 기록
- AI 작업 실패와 재시도 이력 관리
- AI는 승인 판단을 대체하지 않고 초안 생성을 지원

---

## 3.16 No.16 Audit Trail 관리

**영문명:** Audit Trail Management

### 사용 테이블

- `organization`
- `app_user`
- `audit_trail`

### 이 ERD가 설명하는 것

주요 데이터의 생성·수정·삭제와 중요 업무 행위를 변경 전후 값, 수행자, 발생 시점, 변경 사유와 함께 기록한다.

### 중심 테이블: `audit_trail`

`audit_trail`은 다음 정보를 보존한다.

```text
누가        actor_id / actor_type
언제        created_at
무엇을      target_table_name / target_record_id
어떻게      action_type
변경 전     old_values
변경 후     new_values
왜          reason_for_change
어디에서    client_ip / request_uri / user_agent
어떤 요청   request_id / session_id
어떤 버전   target_version / target_revision_number
```

### 테이블별 기능

#### `audit_trail`

- 생성, 수정, 삭제, 내보내기, 로그인, 로그아웃, 리포트 생성, 프로젝트 종료 등의 행위를 기록한다.
- 대상 테이블과 대상 레코드를 다형 참조한다.
- 변경 전후 값을 JSON으로 보존한다.
- 변경 사유, IP 주소, 요청 경로, 사용자 환경을 저장한다.
- 요청 ID와 세션 ID를 통해 하나의 요청에서 발생한 여러 변경을 묶는다.
- 사용자, 시스템, 배치 수행을 구분한다.

#### `app_user`

- 변경을 수행한 사용자를 참조한다.
- 시스템 또는 배치 처리에서는 수행자 ID가 NULL일 수 있다.

#### `organization`

- 수행 사용자의 소속을 기준으로 감사 데이터의 조직 범위를 식별할 수 있다.

### 핵심 설명

> Audit Trail은 현재 데이터를 저장하는 업무 테이블과 별도로 누가 언제 어떤 데이터를 어떻게 변경했고, 왜 변경했는지를 보존합니다. 데이터가 삭제되더라도 삭제 시점의 기존 값을 Audit Trail에 남겨 추적할 수 있습니다.

### 전자서명과의 차이

- **Audit Trail:** 데이터 변경과 중요 시스템 행위의 이력
- **전자서명:** 승인·반려 등 규제상 서명 행위의 증적
- **승인 처리 이력:** Workflow 단계에서 발생한 업무 처리 기록

세 구조를 함께 사용해야 검토·승인 과정과 실제 데이터 변경을 전체적으로 설명할 수 있다.

---

# 4. 전체 발표 연결 순서

16개 ERD는 전체 업무 흐름에 따라 다음 네 개 영역으로 구분된다.

## 4.1 1단계: 플랫폼 기준정보와 업무 시작

대상 ERD:

- No.1 조직·사용자·전역 권한 관리
- No.2 시스템·프로젝트·참여자 관리
- No.3 Validation 활동·선후행 조건 관리

### 영역 설명

> 먼저 조직과 사용자를 구성하고 밸리데이션 대상 시스템에 대해 프로젝트를 생성합니다. 프로젝트별 참여자와 수행 활동을 결정한 후, 선후행 조건에 따라 각 업무를 진행합니다.

## 4.2 2단계: 평가 범위와 설계 산출물 작성

대상 ERD:

- No.4 라이브러리·QIA·공급업체 감사 관리
- No.5 URS·FDS 관리
- No.6 DDS·DQ 관리
- No.7 FRA 위험평가 관리

### 영역 설명

> 프로젝트 초기에는 QIA와 공급업체 감사를 통해 밸리데이션 범위를 확인합니다. 이후 URS를 기준으로 기능 설계와 상세 설계를 작성하고, 설계 적격성과 기능 위험을 평가합니다.

## 4.3 3단계: 시험 수행과 추적성 확보

대상 ERD:

- No.8 IQ·OQ·PQ 적격성 시험 관리
- No.9 설계·위험 산출물 추적성 관리
- No.10 적격성 시험 추적성·RTM 관리
- No.11 VSR·일탈 관리

### 영역 설명

> 승인된 요구사항과 설계를 기준으로 IQ·OQ·PQ 시험을 수행합니다. 모든 요구사항이 설계와 시험으로 연결되는지 RTM으로 확인하고, 발생한 일탈을 해결한 뒤 VSR에서 최종 밸리데이션 결론을 작성합니다.

## 4.4 4단계: 공통 통제와 지원 기능

대상 ERD:

- No.12 Workflow·승인·전자서명 관리
- No.13 파일·증적·파일 정리 관리
- No.14 리포트·알림·백업 운영 관리
- No.15 AI 생성 작업·결과 관리
- No.16 Audit Trail 관리

### 영역 설명

> 전체 업무에는 Workflow, 전자서명, 증적 파일, Audit Trail이 공통적으로 적용됩니다. 리포트, 알림, 백업은 시스템 운영을 지원하고, AI는 산출물 초안 생성을 지원하지만 최종 채택과 승인은 사용자가 수행합니다.

---

# 5. 전체 구조 요약

DVT 플랫폼의 전체 데이터 구조는 다음과 같이 요약할 수 있다.

> 이 데이터 모델은 규제 환경에서 Validation 프로젝트의 전체 생명주기를 관리하기 위한 구조입니다. 조직과 사용자, 대상 시스템과 프로젝트를 기준으로 QIA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR 등의 산출물을 관리합니다. 각 산출물은 프로젝트와 연결되고 상세 항목 간 관계는 공통 추적성 구조로 관리됩니다. 또한 Workflow, 전자서명, 증적 파일, 일탈, Audit Trail을 통해 검토·승인 과정과 데이터 변경 이력을 보존할 수 있도록 구성되어 있습니다. AI 기능은 공식 산출물을 직접 확정하는 구조가 아니라 초안을 생성하고 사용자가 선택·반영하도록 설계되어 있습니다.

---

# 6. 데이터 구조 핵심 메시지

전체 데이터 구조를 이해하기 위한 핵심 메시지는 다음과 같다.

1. **`validation_project`가 대부분의 업무 데이터의 중심이다.**
2. **문서 헤더와 상세 항목을 분리하여 관리한다.**
3. **승인된 기록은 개정 버전을 통해 이전 기록을 보존한다.**
4. **산출물 항목 간 관계는 `traceability_link`로 통합 관리한다.**
5. **RTM은 원천 추적 관계를 집계한 승인 시점의 스냅샷이다.**
6. **Workflow, 승인 처리 이력, 전자서명, Audit Trail은 각각 역할이 다르다.**
7. **파일 자체와 파일의 업무상 증적 연결은 분리되어 있다.**
8. **AI 결과는 공식 산출물이 아니라 사용자 검토 전의 초안이다.**
9. **프로젝트 정상 종료는 단순 진행률이 아니라 활동 승인, 추적성, 일탈 상태를 종합하여 판단한다.**
10. **전체 구조의 목적은 데이터의 일관성, 추적성, 무결성 및 감사 대응력을 확보하는 것이다.**

---

# 7. 주요 용어 구분

## 7.1 `project_member`와 `user_role`

- `user_role`: 시스템 전체에서 사용자에게 직접 부여된 역할
- `project_member`: 특정 프로젝트에서 사용자에게 부여된 역할

## 7.2 `group_role`과 `project_member`

- `group_role`: 사용자 그룹 단위로 전역 또는 프로젝트 범위 역할을 일괄 부여
- `project_member`: 프로젝트에 실제 참여하는 개별 사용자와 역할을 관리

## 7.3 `traceability_link`와 `rtm_item`

- `traceability_link`: 현재 운영 중인 원천 추적 관계
- `rtm_item`: RTM 생성 또는 승인 시점의 상세 추적 결과 스냅샷

## 7.4 `workflow_instance`, `approval_action`, `electronic_signature`, `audit_trail`

- `workflow_instance`: 문서 검토·승인 절차의 전체 진행 상태
- `workflow_step`: Workflow의 개별 검토·승인 단계
- `approval_action`: 실제 상신·검토·승인·반려 처리 이력
- `electronic_signature`: 처리 행위와 연결된 전자서명 증적
- `audit_trail`: 데이터 변경 및 중요 시스템 행위의 감사 이력

## 7.5 `file_asset`과 `evidence_link`

- `file_asset`: 파일명, 경로, 크기, 유형 등 파일 자체의 메타데이터
- `evidence_link`: 파일이 어떤 문서 또는 시험 항목의 증적인지 나타내는 업무 관계

## 7.6 `protocol_status`와 `record_status`

- `protocol_status`: 시험을 수행하기 전에 시험 절차가 승인되었는지 나타내는 상태
- `record_status`: 승인된 절차로 시험을 수행한 후 실제 결과 기록이 승인되었는지 나타내는 상태

## 7.7 정상종료와 강제종료

- 정상종료: 선택된 활동의 승인, 요구사항 추적성, 위험 커버리지, 일탈 종결 등 정상 완료 조건을 충족한 종료
- 강제종료: 정상 완료 조건과 별개로 업무상 사유에 의해 중단하며 종료 유형, 사유, 요청자와 시점을 기록하는 종료

---

## 문서 구성 기준

각 ERD는 다음 순서에 따라 검토할 수 있다.

1. 이 ERD가 담당하는 업무 범위
2. 중심 테이블
3. 주요 상세 테이블
4. 테이블 간 관계
5. 상태와 버전 관리 방식
6. 다른 ERD 영역과 연결되는 지점
7. 규제 준수 관점의 핵심 의미

이 문서는 각 ERD 화면과 함께 활용할 수 있는 **공식 데이터 구조 설명 및 발표 참고자료**이다.
