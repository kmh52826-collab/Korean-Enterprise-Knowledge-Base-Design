<a id="top"></a>

# ERD Specifications

Validation Management Platform의 72개 테이블을 14개 업무 영역으로 나누어 설명합니다. 각 영역에서 관리하는 정보와 주요 테이블의 연결을 확인할 수 있습니다.

> [!NOTE]
> 이 문서는 데이터 모델링 및 시스템 설계를 설명하기 위한 자료입니다. 예시값은 비식별 샘플을 사용하며, 특정 조직의 실제 운영 데이터를 나타내지 않습니다.

관계 구조는 주요 연결을 간단히 보여주는 도식입니다. 공통으로 사용하는 테이블은 여러 ERD에 함께 표시될 수 있습니다. 상세 구조는 각 영역의 ERD 링크에서, 컬럼 정의와 업무 규칙은 [Data Dictionary](./data-dictionary.md)에서 확인하세요.

## 목차

1. [조직·사용자·권한](#erd-01)
2. [시스템 인벤토리·프로젝트 관리](#erd-02)
3. [검증 계획·사전 평가](#erd-03)
4. [요구사항·설계·위험평가](#erd-04)
5. [IQ 설치 적격성 시험](#erd-05)
6. [OQ 운전 적격성 시험](#erd-06)
7. [PQ 성능 적격성 시험](#erd-07)
8. [일탈·조치·재수행](#erd-08)
9. [요구사항·시험 추적성](#erd-09)
10. [VSR 종합 보고](#erd-10)
11. [결재·전자서명·감사기록](#erd-11)
12. [산출물·문서 개정·파일](#erd-12)
13. [라이브러리·규정 근거](#erd-13)
14. [AI 생성·적용](#erd-14)

---

<a id="erd-01"></a>

## 1. 조직·사용자·권한

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/01-organization-security>

**테이블 수: 10개**

### 관계 구조

```text
organization
  ├─ app_user ─ user_role ─ role
  └─ user_group
       ├─ user_group_member ─ app_user
       └─ group_role ─ role

app_user / user_group
  ├─ access_permission_grant (화면·프로젝트 접근권한)
  └─ inventory_role_grant (인벤토리 업무 권한)
```

### 구조 개요

조직별 사용자와 그룹을 관리하고, 개인·그룹에 업무 역할과 접근권한을 부여합니다. 작성자·검토자·승인자 같은 역할, 화면과 프로젝트의 접근권한, 인벤토리 업무 권한을 구분해 관리하는 구조입니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `organization` | 사용자와 그룹이 소속되는 조직 정보 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `role` | 작성자·검토자·승인자 등 업무 역할의 정의 |
| `user_role` | 사용자에게 직접 부여한 업무 역할 |
| `user_group` | 조직별 사용자 그룹 |
| `user_group_member` | 그룹에 소속된 사용자와 참여 상태 |
| `group_role` | 그룹에 부여한 공통 또는 프로젝트별 업무 역할 |
| `access_permission_grant` | 개인·그룹의 화면 및 프로젝트 접근권한 |
| `inventory_role_grant` | 개인·그룹의 인벤토리 작성·검토·승인·폐기 권한 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |

### 핵심 사항

- 관리 권한등급과 업무 역할은 별개입니다. 계정의 관리 권한만으로 검토자나 승인자가 되지는 않습니다.
- 사용자에게 직접 부여한 권한과 소속 그룹에서 받은 권한을 함께 적용합니다. 그룹 권한은 활성 그룹과 구성원을 기준으로 판단합니다.
- 편집·폐기에는 접근권한과 업무 역할이 모두 필요합니다. 검토·승인에는 결재 담당자 또는 대체 담당자로 배정되어 있어야 합니다.

[↑ 목차로](#top)

---

<a id="erd-02"></a>

## 2. 시스템 인벤토리·프로젝트 관리

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/02-inventory-project>

**테이블 수: 14개**

### 관계 구조

```text
system_asset (검증 대상 시스템)
  ├─ system_asset_revision (개정·승인 이력)
  └─ validation_project
       ├─ project_system_baseline (채택한 승인 개정)
       ├─ project_member (참여자)
       ├─ project_activity ─ validation_activity
       └─ project_closure_request (종료 요청)

validation_activity ─ activity_dependency (활동 선후행 조건)
```

### 구조 개요

검증 대상 시스템의 현재 정보와 개정 이력을 관리하고, 이를 기준으로 프로젝트를 운영합니다. 프로젝트에는 검증 기준으로 채택한 승인 개정, 참여자, 수행 활동, 종료 요청을 연결합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `system_asset` | 검증 대상 시스템·장비의 현재 정보와 승인 상태 |
| `system_asset_revision` | 시스템의 변경·승인 이력과 당시 정보 원문 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_system_baseline` | 프로젝트가 검증 기준으로 채택한 시스템 승인 개정 |
| `project_member` | 프로젝트 참여자와 담당 역할 |
| `validation_activity` | 검증 계획·평가·시험 등 수행 활동의 공통 목록 |
| `project_activity` | 프로젝트에서 선택한 활동과 진행 상태 |
| `activity_dependency` | 활동 사이의 선행·후행 조건 |
| `project_closure_request` | 프로젝트 종료 요청과 회차별 승인 이력 연결 |
| `organization` | 사용자와 그룹이 소속되는 조직 정보 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `role` | 작성자·검토자·승인자 등 업무 역할의 정의 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |

### 핵심 사항

- 시스템의 현재 정보와 프로젝트가 채택한 검증 기준을 구분합니다. 시스템이 나중에 개정되어도 프로젝트가 사용한 승인 원문은 보존합니다.
- 프로젝트는 선택한 활동과 생성 당시 채택한 선후행 조건에 따라 진행됩니다. 공통 조건을 변경해도 기존 프로젝트에 자동 반영하지 않습니다.
- 종료 요청은 회차별로 관리합니다. 반려 후 다시 요청하더라도 이전 요청의 사유와 승인 이력을 확인할 수 있습니다.

[↑ 목차로](#top)

---

<a id="erd-03"></a>

## 3. 검증 계획·사전 평가

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/03-planning-assessment>

**테이블 수: 10개**

### 관계 구조

```text
validation_project
  ├─ vp_plan (검증 계획)
  │    └─ vp_section (목차·본문)
  ├─ qia_assessment (품질 영향 평가)
  │    └─ qia_module_item (모듈)
  │         └─ qia_process (프로세스)
  └─ vendor_audit (공급업체 감사)
```

### 구조 개요

프로젝트의 검증 계획(VP), 품질 영향 평가(QIA), 공급업체 감사(VA)를 관리합니다. VP는 목차와 본문으로, QIA는 공통 평가·모듈·프로세스로 구성하며, 공급업체 감사에는 감사자·일자와 평가 파일을 기록합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `vp_plan` | 검증 계획의 개정과 승인 상태 |
| `vp_section` | 검증 계획에 포함된 목차와 본문 |
| `qia_assessment` | 프로젝트 공통 Part 11 평가와 판정 기준 |
| `qia_module_item` | 모듈별 품질 영향 평가와 개정·승인 상태 |
| `qia_process` | 모듈에 속한 프로세스별 GxP 평가 응답과 판정 기준 |
| `vendor_audit` | 공급업체 감사의 개정, 감사자·일자와 평가 파일 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |

### 핵심 사항

- VP·QIA 모듈·공급업체 감사는 각각 개정과 승인을 관리합니다. 승인 당시의 본문과 평가 응답을 보존합니다.
- QIA는 응답뿐 아니라 당시 사용한 질문과 판정 기준도 함께 보존합니다. 이후 기준이 바뀌어도 과거 결과를 해석할 수 있습니다.
- 프로세스가 없는 QIA 모듈은 NON_GXP로 표시하며 승인할 수 있습니다. 이 표시는 프로세스 평가가 완료되었다는 뜻은 아닙니다.

[↑ 목차로](#top)

---

<a id="erd-04"></a>

## 4. 요구사항·설계·위험평가

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/04-requirements-design-risk>

**테이블 수: 15개**

### 관계 구조

```text
validation_project
  ├─ requirement (URS)
  ├─ fds_spec (FDS)
  ├─ dds_spec (DDS)
  ├─ dq_assessment ─ dq_item (설계 적격성 평가)
  └─ fra_assessment ─ fra_item (기능 위험평가)

dq_item ─ requirement / fds_spec / dds_spec
fra_item ─ requirement
```

### 구조 개요

URS는 검증할 요구사항을, FDS·DDS는 기능 설계와 상세 설계 문서를 관리합니다. DQ는 요구사항과 설계의 적합성을 평가하고, FRA는 요구사항별 위험과 대응 근거를 기록합니다. 규정 조항과 라이브러리는 작성·평가의 근거로 연결합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `requirement` | 요구사항의 내용·수용 기준과 개정·승인 상태 |
| `fds_spec` | 기능 설계 문서의 파일과 개정·승인 상태 |
| `dds_spec` | 상세 설계 문서의 파일과 개정·승인 상태 |
| `dq_assessment` | 프로젝트의 설계 적격성 평가 정보 |
| `dq_item` | 요구사항·설계 문서에 대한 적합성 판정과 검토 내용 |
| `fra_assessment` | 프로젝트의 기능 위험평가 정보 |
| `fra_item` | 요구사항별 위험 시나리오, 평가 결과와 SOP 근거 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `library_item` | 요구사항·위험평가·시험 작성에 재사용할 항목 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |
| `regulatory_clause` | 규정 문서의 개별 조항과 원문 위치 |
| `requirement_regulation` | 요구사항과 규정 조항의 연결 및 적용 근거 |
| `regulatory_source` | 규정·가이드라인·내부 SOP의 문서와 판본 |

### 핵심 사항

- FDS와 DDS는 각각 독립된 문서로 관리합니다. 파일을 교체하면 새 문서 개정을 만들고 기존 원본은 보존합니다.
- DQ 항목은 URS 한 개정에 FDS·DDS를 각각 최대 한 개정 연결합니다. 승인 요청에는 같은 프로젝트의 유효한 승인 설계 문서가 하나 이상 필요합니다.
- DQ·FRA는 평가에 사용한 정확한 요구사항·설계 개정을 보존합니다. FRA의 판정 기준과 승인 당시 위험평가 결과도 함께 유지합니다.

[↑ 목차로](#top)

---

<a id="erd-05"></a>

## 5. IQ 설치 적격성 시험

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/05-iq-testing>

**테이블 수: 13개**

### 관계 구조

```text
validation_project
  └─ iq_assessment (IQ 평가)
       └─ iq_item (시험 항목·프로토콜)
            ├─ iq_step (시험 절차)
            └─ iq_execution (수행 회차)
                 └─ iq_step_execution (절차별 결과)
```

### 구조 개요

IQ는 설치 적격성을 확인할 시험 내용과 실제 수행 결과를 관리합니다. 평가 아래에 시험 항목과 세부 절차를 구성하고, 수행 회차마다 판정과 절차별 완료 기록을 남깁니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `iq_assessment` | IQ 평가의 개정, 시험 구성과 승인 상태 요약 |
| `iq_item` | 설치 적격성 시험의 내용·기준과 프로토콜 개정 |
| `iq_step` | IQ 시험 항목의 세부 절차와 순서 |
| `iq_execution` | IQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `iq_step_execution` | IQ 수행에서 각 절차의 완료 여부와 처리 기록 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_system_baseline` | 프로젝트가 검증 기준으로 채택한 시스템 승인 개정 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `library_item` | 요구사항·위험평가·시험 작성에 재사용할 항목 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `evidence_link` | 업무 기록과 증적 파일의 연결 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |

### 핵심 사항

- 평가의 시험 구성과 개별 시험의 개정을 구분합니다. 평가가 개정되어도 내용이 바뀌지 않은 시험은 다시 포함할 수 있습니다.
- 승인된 프로토콜로 시험을 수행합니다. 시험 판정과 수행 결과의 승인은 별도로 관리합니다.
- 재수행은 새 회차로, 기존 결과의 정정은 같은 회차의 수정 이력으로 남깁니다. 과거 결과·증적과 수행 당시 시스템 기준은 보존합니다.

[↑ 목차로](#top)

---

<a id="erd-06"></a>

## 6. OQ 운전 적격성 시험

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/06-oq-testing>

**테이블 수: 13개**

### 관계 구조

```text
validation_project
  └─ oq_assessment (OQ 평가)
       └─ oq_item (시험 항목·프로토콜)
            ├─ oq_step (시험 절차)
            └─ oq_execution (수행 회차)
                 └─ oq_step_execution (절차별 결과)
```

### 구조 개요

OQ는 운전 적격성 시험의 내용·기대 결과·허용 기준과 실제 수행 결과를 관리합니다. IQ와 같은 구성으로 평가, 시험 항목, 세부 절차, 수행 기록을 나누되 OQ의 데이터와 승인 상태를 별도로 관리합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `oq_assessment` | OQ 평가의 개정, 시험 구성과 승인 상태 요약 |
| `oq_item` | 운전 적격성 시험의 내용·기준과 프로토콜 개정 |
| `oq_step` | OQ 시험 항목의 세부 절차와 순서 |
| `oq_execution` | OQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `oq_step_execution` | OQ 수행에서 각 절차의 완료 여부와 처리 기록 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_system_baseline` | 프로젝트가 검증 기준으로 채택한 시스템 승인 개정 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `library_item` | 요구사항·위험평가·시험 작성에 재사용할 항목 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `evidence_link` | 업무 기록과 증적 파일의 연결 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |

### 핵심 사항

- 시험 내용이 바뀌면 새 프로토콜 개정을 작성합니다. 기존 수행 기록은 당시 사용한 프로토콜을 계속 참조합니다.
- 시험 판정과 결과 승인 상태를 구분합니다. 재수행과 결과 정정도 서로 다른 이력으로 남깁니다.
- IQ·PQ와의 선후행은 프로젝트 활동에서 관리합니다. 요구사항과의 연결은 추적성 영역에서, 시험 실패에 대한 조치는 일탈 영역에서 확인합니다.

[↑ 목차로](#top)

---

<a id="erd-07"></a>

## 7. PQ 성능 적격성 시험

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/07-pq-testing>

**테이블 수: 13개**

### 관계 구조

```text
validation_project
  └─ pq_assessment (PQ 평가)
       └─ pq_item (시험 항목·프로토콜)
            ├─ pq_step (시험 절차)
            └─ pq_execution (수행 회차)
                 └─ pq_step_execution (절차별 결과)
```

### 구조 개요

PQ는 성능 적격성 시험의 구성과 실제 수행 결과를 관리합니다. 평가별 시험 목록, 시험 내용과 절차, 회차별 판정과 완료 기록을 구분하여 보존합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `pq_assessment` | PQ 평가의 개정, 시험 구성과 승인 상태 요약 |
| `pq_item` | 성능 적격성 시험의 내용·기준과 프로토콜 개정 |
| `pq_step` | PQ 시험 항목의 세부 절차와 순서 |
| `pq_execution` | PQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `pq_step_execution` | PQ 수행에서 각 절차의 완료 여부와 처리 기록 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_system_baseline` | 프로젝트가 검증 기준으로 채택한 시스템 승인 개정 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `library_item` | 요구사항·위험평가·시험 작성에 재사용할 항목 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `evidence_link` | 업무 기록과 증적 파일의 연결 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |

### 핵심 사항

- 평가 구성, 시험 프로토콜, 수행 결과는 각각 별도로 관리합니다. 같은 승인 프로토콜을 여러 회차에 수행할 수 있습니다.
- 결과 등록에는 절차 완료와 수행 서명이 필요합니다. 등록 후 수정은 원래 결과와 증적을 보존하는 정정 절차로 처리합니다.
- 프로젝트의 현재 시스템 기준이 바뀌어도 과거 수행의 기준은 유지합니다. 승인된 수행 결과는 VSR 종합 보고의 근거가 됩니다.

[↑ 목차로](#top)

---

<a id="erd-08"></a>

## 8. 일탈·조치·재수행

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/08-deviation-rerun>

**테이블 수: 9개**

### 관계 구조

```text
iq_execution / oq_execution / pq_execution (실패한 시험 수행)
    ↓ 일탈 등록
deviation
  └─ deviation_action_round (회차별 조치·승인)
       ↓ 재수행
iq_execution / oq_execution / pq_execution (새 수행 기록)

재실패 시 조치 회차 추가
```

### 구조 개요

시험 실패로 발생한 일탈을 등록하고, 조치 승인부터 재수행과 최종 종료까지 관리합니다. `deviation`은 일탈의 전체 상태를, `deviation_action_round`는 회차별 조치 내용과 승인·재수행 이력을 관리합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `deviation` | 시험 실패로 발생한 일탈과 전체 처리 상태 |
| `deviation_action_round` | 회차별 조치 내용, 승인 이력과 재수행 연결 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `iq_execution` | IQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `oq_execution` | OQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `pq_execution` | PQ 수행 회차별 판정, 결과 정정과 승인 기록 |

### 핵심 사항

- 최초 실패 기록을 보존하고, 각 조치를 승인 근거와 재수행 결과에 연결합니다. 재수행에서 다시 실패하면 다음 조치 회차를 추가합니다.
- 같은 조치를 반려 후 수정하는 경우에는 해당 회차의 새 개정으로 관리합니다. 승인과 서명은 당시의 조치 원문에 연결됩니다.
- 재수행 성공 후 완료보고를 승인하는 경로와, 조치 승인 후 재수행 없이 사유를 남기고 종료하는 경로를 구분합니다. 각 종료 경로에 맞는 최종 서명을 보존합니다.

[↑ 목차로](#top)

---

<a id="erd-09"></a>

## 9. 요구사항·시험 추적성

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/09-traceability>

**테이블 수: 11개**

### 관계 구조

```text
requirement (URS)
  └─ traceability_link (추적 관계)
       ├─ fds_spec / dds_spec (설계)
       ├─ dq_item / fra_item (설계·위험평가)
       └─ iq_item / oq_item / pq_item (시험 항목)

추적 관계와 시험 결과 → RTM 대시보드
```

### 구조 개요

요구사항이 어떤 설계·평가·시험으로 이어지는지 `traceability_link`로 연결합니다. RTM 대시보드는 이 관계와 실제 시험 결과를 함께 조회하여 요구사항별 검증 현황을 보여줍니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `traceability_link` | 요구사항·설계·평가·시험 사이의 추적 관계 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `requirement` | 요구사항의 내용·수용 기준과 개정·승인 상태 |
| `fds_spec` | 기능 설계 문서의 파일과 개정·승인 상태 |
| `dds_spec` | 상세 설계 문서의 파일과 개정·승인 상태 |
| `dq_item` | 요구사항·설계 문서에 대한 적합성 판정과 검토 내용 |
| `fra_item` | 요구사항별 위험 시나리오, 평가 결과와 SOP 근거 |
| `iq_item` | 설치 적격성 시험의 내용·기준과 프로토콜 개정 |
| `oq_item` | 운전 적격성 시험의 내용·기준과 프로토콜 개정 |
| `pq_item` | 성능 적격성 시험의 내용·기준과 프로토콜 개정 |

### 핵심 사항

- 추적 관계는 연결 당시의 정확한 업무 개정을 가리킵니다. 새 개정이 생겨도 과거 승인 근거의 연결을 자동으로 바꾸지 않습니다.
- 하나의 시험에 여러 요구사항을 연결할 수 있습니다. 시험과 연결되어 있는지, 실제 시험이 성공하고 승인되었는지는 별도로 확인합니다.
- RTM은 대시보드 조회 기능으로 운영합니다. 독립 수행 단계나 승인 문서로 관리하지 않으며, 프로젝트 진행률과 VSR의 활동 목록에도 별도 항목으로 넣지 않습니다.

[↑ 목차로](#top)

---

<a id="erd-10"></a>

## 10. VSR 종합 보고

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/10-vsr-summary>

**테이블 수: 14개**

### 관계 구조

```text
validation_project
  └─ vsr_assessment (종합 보고)
       └─ vsr_item (활동별 결과 요약)
            ├─ project_activity (수행 활동)
            └─ deliverable_revision (근거 문서)

계획·평가·시험 결과 → VSR 집계·승인 → 프로젝트 종료 판단
```

### 구조 개요

프로젝트의 계획·평가·시험 결과를 모아 밸리데이션 종합 보고서(VSR)를 구성합니다. `vsr_assessment`는 종합 결론과 확인 상태를, `vsr_item`은 활동별 결과와 그 근거를 관리합니다. 승인된 보고서에는 집계 당시의 결과와 승인정보를 함께 보존합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `vsr_assessment` | 종합 보고의 개정, 결론과 최종 확인 상태 |
| `vsr_item` | 수행 활동별 결과 요약과 집계 당시 근거 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_activity` | 프로젝트에서 선택한 활동과 진행 상태 |
| `deliverable_revision` | 문서 개정별 내용·근거·승인 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `vp_plan` | 검증 계획의 개정과 승인 상태 |
| `qia_module_item` | 모듈별 품질 영향 평가와 개정·승인 상태 |
| `vendor_audit` | 공급업체 감사의 개정, 감사자·일자와 평가 파일 |
| `iq_execution` | IQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `oq_execution` | OQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `pq_execution` | PQ 수행 회차별 판정, 결과 정정과 승인 기록 |

### 핵심 사항

- 프로젝트에서 선택한 수행 활동을 집계하며, RTM과 VSR 자체는 활동별 상세 목록에서 제외합니다. ERD의 계획·평가·시험 테이블은 집계 근거를 보여주는 대표 사례입니다.
- VSR의 업무 확인과 출력 문서의 승인은 별도로 관리합니다. 업무 확인을 마쳤어도 출력 문서가 아직 생성되지 않았을 수 있습니다.
- 승인 이후 원본 결과가 바뀌면 새 VSR 개정에서 다시 집계하고 확인받습니다. 과거 보고서의 결과와 근거는 그대로 유지합니다.

[↑ 목차로](#top)

---

<a id="erd-11"></a>

## 11. 결재·전자서명·감사기록

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/11-workflow-signature-audit>

**테이블 수: 15개**

### 관계 구조

```text
project_workflow_config (결재선 설정)
  └─ workflow_instance (결재 실행)
       └─ workflow_step (검토·승인 단계)
            ├─ workflow_step_assignee (담당자 배정)
            └─ approval_action (처리 이력)
                 └─ electronic_signature (전자서명)

audit_trail (업무 전반의 감사기록)
```

### 구조 개요

여러 업무에서 사용하는 검토·승인 절차와 담당자, 처리 결과를 공통으로 관리합니다. 결재선 설정을 바탕으로 개별 결재를 진행하고, 각 단계의 배정과 처리 이력을 남깁니다. 전자서명은 서명 대상 원문을, 감사기록은 변경·접속·내보내기 등의 행위를 보존합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `workflow_step` | 결재 건의 검토·승인 단계와 진행 순서 |
| `workflow_step_assignee` | 결재 단계별 담당자와 대체 담당자 배정 |
| `approval_action` | 상신·검토·승인·반려·취소의 처리 이력 |
| `project_workflow_config` | 프로젝트 공통 또는 활동별 결재선 설정 |
| `electronic_signature` | 서명자, 서명 대상과 서명 당시 원문 |
| `audit_trail` | 사용자·시스템의 행위와 데이터 변경 이력 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_activity` | 프로젝트에서 선택한 활동과 진행 상태 |
| `requirement` | 요구사항의 내용·수용 기준과 개정·승인 상태 |
| `deliverable_revision` | 문서 개정별 내용·근거·승인 상태 |
| `deviation_action_round` | 회차별 조치 내용, 승인 이력과 재수행 연결 |
| `system_asset` | 검증 대상 시스템·장비의 현재 정보와 승인 상태 |
| `project_closure_request` | 프로젝트 종료 요청과 회차별 승인 이력 연결 |

### 핵심 사항

- 결재선 설정과 실제 결재 건을 구분합니다. 개별 상신 건은 당시 적용한 경로와 담당 배정에 따라 검토·승인 이력을 남깁니다.
- 전자서명은 정확한 대상과 개정에 연결되며, 서명한 원문도 함께 보존합니다. 서명 이후 본문을 바꾸려면 새 개정과 새 서명이 필요합니다.
- 승인 여부와 담당자 처리는 결재기록에서, 업무 전반의 행위는 감사기록에서 확인합니다. 감사기록은 기존 내용을 고치지 않고 새 기록을 추가합니다.

[↑ 목차로](#top)

---

<a id="erd-12"></a>

## 12. 산출물·문서 개정·파일

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/12-documents-files>

**테이블 수: 15개**

### 관계 구조

```text
deliverable_document (산출물)
  └─ deliverable_revision (문서 개정)
       ├─ deliverable_section (목차·본문)
       └─ report_generation (PDF 생성)
            └─ file_asset (결과 파일)

업무 기록 ─ evidence_link (증적 연결) ─ file_asset
```

### 구조 개요

프로젝트 산출물을 문서, 개정, 목차·본문으로 나누어 관리하고 PDF 생성과 증적 파일을 연결합니다. 각 문서 개정에는 작성 근거가 된 업무 내용과 당시의 결과를 보존합니다. 이 ERD는 IQ를 예시로 문서 근거와 시험 증적의 연결을 보여줍니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `deliverable_document` | 산출물의 문서번호, 종류와 소속 활동 |
| `deliverable_revision` | 문서 개정별 내용·근거·승인 상태 |
| `deliverable_section` | 문서 개정에 포함된 목차와 본문 |
| `report_generation` | PDF 생성 요청, 진행 상태와 결과 파일 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |
| `evidence_link` | 업무 기록과 증적 파일의 연결 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `project_activity` | 프로젝트에서 선택한 활동과 진행 상태 |
| `project_system_baseline` | 프로젝트가 검증 기준으로 채택한 시스템 승인 개정 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `workflow_instance` | 특정 업무에 대한 결재 진행 건 |
| `iq_assessment` | IQ 평가의 개정, 시험 구성과 승인 상태 요약 |
| `iq_item` | 설치 적격성 시험의 내용·기준과 프로토콜 개정 |
| `iq_execution` | IQ 수행 회차별 판정, 결과 정정과 승인 기록 |
| `iq_step_execution` | IQ 수행에서 각 절차의 완료 여부와 처리 기록 |

### 핵심 사항

- 문서 개정과 원본 업무의 개정·승인은 별도로 관리합니다. 문서만 바뀔 때 변경 없는 업무 원본을 재사용할 수 있으며, 문서 승인이 원본 업무의 승인을 대신하지 않습니다.
- 승인된 문서의 본문·표·근거는 보존하고, 변경이 필요하면 새 문서 개정을 작성합니다. 이후 원본이나 검증 대상 기준이 바뀌어도 과거 문서가 사용한 기준은 유지합니다.
- 증적 파일은 해당 시험 수행이나 절차 기록에 연결하며, 파일이 바뀌어도 승인에 사용한 원본은 보존합니다. PDF 생성 작업과 생성된 파일의 보관 정보는 따로 관리합니다.

[↑ 목차로](#top)

---

<a id="erd-13"></a>

## 13. 라이브러리·규정 근거

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/13-library-regulation>

**테이블 수: 12개**

### 관계 구조

```text
regulatory_source (규정 문서판)
  └─ regulatory_clause (규정 조항)
       ├─ requirement_regulation ─ requirement (URS)
       └─ library_item_regulation ─ library_item (재사용 항목)

library_item
  └─ URS·FRA·IQ·OQ·PQ 작성에 활용
```

### 구조 개요

라이브러리는 요구사항·위험평가·시험을 작성할 때 재사용하는 항목을 제공합니다. 규정 근거는 문서의 판본과 개별 조항으로 나누어 관리하며, 요구사항과 라이브러리 항목에 필요한 조항을 연결합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `library_item` | 요구사항·위험평가·시험 작성에 재사용할 항목 |
| `regulatory_source` | 규정·가이드라인·내부 SOP의 문서와 판본 |
| `regulatory_clause` | 규정 문서의 개별 조항과 원문 위치 |
| `requirement_regulation` | 요구사항과 규정 조항의 연결 및 적용 근거 |
| `library_item_regulation` | 라이브러리 항목과 규정 조항의 연결 및 적용 근거 |
| `file_asset` | 파일의 저장 위치와 원본을 식별하는 정보 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `requirement` | 요구사항의 내용·수용 기준과 개정·승인 상태 |
| `fra_item` | 요구사항별 위험 시나리오, 평가 결과와 SOP 근거 |
| `iq_item` | 설치 적격성 시험의 내용·기준과 프로토콜 개정 |
| `oq_item` | 운전 적격성 시험의 내용·기준과 프로토콜 개정 |
| `pq_item` | 성능 적격성 시험의 내용·기준과 프로토콜 개정 |

### 핵심 사항

- 라이브러리에서 가져온 내용은 프로젝트의 독립된 업무 항목으로 관리합니다. 이후 라이브러리 변경이 자동 반영되지 않으며, URS를 가져올 때는 규정 근거도 함께 복사합니다.
- 규정이 개정되면 새 판본으로 등록합니다. 승인에 사용한 판본·조항·인용 정보는 보존하며, 이후 조항이 비활성화되어도 과거 연결은 유지합니다.
- 하나의 요구사항이나 라이브러리 항목에 여러 규정 조항을 연결할 수 있습니다. FRA의 SOP 근거는 내부 SOP에 해당하는 조항을 직접 연결합니다.

[↑ 목차로](#top)

---

<a id="erd-14"></a>

## 14. AI 생성·적용

ERD 링크 : <https://drawsql.app/teams/minho-kim/diagrams/14-ai-generation>

**테이블 수: 12개**

### 관계 구조

```text
ai_generation_job (생성 요청)
  └─ ai_generation_result (결과 묶음)
       └─ ai_result_item (개별 초안)

사용자 검토·선택 → 업무 항목·문서 본문에 적용 → 검토·승인
```

### 구조 개요

AI 생성 요청부터 사용자 검토와 실제 업무 반영까지의 과정을 관리합니다. 생성 요청, 결과 묶음, 개별 초안을 구분하고, 채택한 초안이 어떤 요구사항·위험평가·시험·문서 본문에 적용되었는지 기록합니다.

### 테이블별 역할

| 테이블 | 역할 |
|---|---|
| `ai_generation_job` | AI 생성 요청, 입력 조건과 진행 상태 |
| `ai_generation_result` | 생성 결과 묶음과 사용자 선택·반영 여부 |
| `ai_result_item` | 개별 초안과 실제 업무에 적용한 결과 |
| `validation_project` | 검증 프로젝트의 기본 정보와 진행·종료 상태 |
| `app_user` | 사용자 계정, 소속 조직, 계정 상태와 관리 권한 |
| `requirement` | 요구사항의 내용·수용 기준과 개정·승인 상태 |
| `fra_item` | 요구사항별 위험 시나리오, 평가 결과와 SOP 근거 |
| `iq_item` | 설치 적격성 시험의 내용·기준과 프로토콜 개정 |
| `oq_item` | 운전 적격성 시험의 내용·기준과 프로토콜 개정 |
| `pq_item` | 성능 적격성 시험의 내용·기준과 프로토콜 개정 |
| `deliverable_revision` | 문서 개정별 내용·근거·승인 상태 |
| `deliverable_section` | 문서 개정에 포함된 목차와 본문 |

### 핵심 사항

- 생성 완료, 사용자 선택, 업무 적용은 별도 상태입니다. AI가 결과를 만들었다는 사실만으로 업무에 반영되거나 승인된 것으로 보지 않습니다.
- 새 항목은 적용 전에 미리보기로 검토할 수 있으며, 적용 후 실제 업무 항목과 연결합니다. 문서 생성은 문서 개정을 대상으로 요청하고, 결과는 해당 목차의 본문에 반영합니다.
- AI 결과는 편집 가능한 초안이나 새 개정에 적용합니다. 승인된 원문을 덮어쓰지 않으며, 적용한 내용도 일반적인 검토·승인 절차를 거칩니다.

[↑ 목차로](#top)
