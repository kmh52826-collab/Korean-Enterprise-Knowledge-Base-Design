# 데이터 테이블 정의서

> Validation Management Platform 데이터 모델 문서  
> **보안 처리**: 예시값은 문서 공개를 고려하여 비식별 샘플 값으로 치환

## 목차

- [1. 문서 개요](#1-문서-개요)
- [2. 표기 기준](#2-표기-기준)
- [3. 테이블 목록](#3-테이블-목록)
- [4. 도메인별 바로가기](#4-도메인별-바로가기)
- [5. 상세 테이블 및 컬럼 정의](#5-상세-테이블-및-컬럼-정의)

## 1. 문서 개요

- 전체 테이블: **51개**
- 전체 컬럼: **783개**
- 도메인: **25개**
- 명명 규칙: 테이블 및 컬럼은 `snake_case`, PK는 엔터티별 식별자 컬럼 사용
- 날짜/시간 기준: DB에는 UTC 저장, 화면에서는 사용자 또는 사업장 Timezone 적용
- GxP 원칙: 승인 기록은 직접 수정하지 않고 Revision 및 상태 이력으로 관리하며 주요 변경은 Audit Trail에 기록

## 2. 표기 기준

| 표기 | 의미 |
|---|---|
| `Y` | 적용 또는 필수 |
| `N` | 미적용 또는 선택 |
| `-` | 해당 없음 또는 미지정 |
| PK | Primary Key |
| FK | Foreign Key |
| Not Null | NULL 허용 여부의 반대 조건 |
| Audit | Audit Trail 기록 대상 여부 |

## 3. 테이블 목록

| No | Domain | 논리 테이블명 | 물리 테이블명 | PK | 주요 참조(FK) | GxP | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | 조직/고객사 | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | 사용자 | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | 역할 | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | 사용자 역할 | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id` | High | Y |
| 5 | Compliance | 전자서명 | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 6 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 7 | File | 파일 자산 | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 8 | System | 시스템/장비 식별 정보 | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 9 | Library | 라이브러리 항목 마스터 | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 10 | Validation | Validation 프로젝트 | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by` | High | Y |
| 11 | QIA | 품질 영향 평가 헤더 | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id` | High | Y |
| 12 | QIA | QIA 모듈 상세 평가 | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id` | High | Y |
| 13 | VA | 공급업체 감사 평가 | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id` | High | Y |
| 14 | URS | 사용자 요구사항 명세 | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by` | High | Y |
| 15 | FDS | 기능 설계 명세서 | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by` | High | Y |
| 16 | FDS | FDS 상세 항목 | [`fds_item`](#table-fds_item) | `fds_item_id` | `fds_id, created_by, updated_by` | High | Y |
| 17 | FDS | FDS 인터페이스 정의 | [`fds_interface`](#table-fds_interface) | `fds_interface_id` | `fds_id, created_by, updated_by` | High | Y |
| 18 | DQ | 설계 적격성 평가 | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 19 | DQ | DQ 상세 평가 항목 | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, reviewed_by, created_by, updated_by` | High | Y |
| 20 | FRA | 기능 위험평가 | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 21 | FRA | FRA 위험 상세 항목 | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by` | High | Y |
| 22 | IQ | 설치 적격성 평가 | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 23 | IQ | IQ 상세 테스트 항목 | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, executed_by, created_by, updated_by` | High | Y |
| 24 | OQ | 운전 적격성 평가 | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 25 | OQ | OQ 상세 테스트 항목 | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, executed_by, created_by, updated_by` | High | Y |
| 26 | PQ | 성능 적격성 평가 | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 27 | PQ | PQ 상세 테스트 항목 | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, executed_by, created_by, updated_by` | High | Y |
| 28 | RTM | 요구사항 추적 매트릭스 | [`rtm_assessment`](#table-rtm_assessment) | `rtm_id` | `project_id, created_by, updated_by` | Critical | Y |
| 29 | RTM | RTM 상세 추적 항목 | [`rtm_item`](#table-rtm_item) | `rtm_item_id` | `rtm_id, requirement_id, created_by, updated_by` | Critical | Y |
| 30 | VSR | 밸리데이션 종합 보고서 | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by` | Critical | Y |
| 31 | VSR | VSR 활동 요약 항목 | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by` | Critical | Y |
| 32 | Workflow | Workflow 인스턴스 | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 33 | Workflow | Workflow 단계 | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, assignee_id, created_by, updated_by` | Critical | Y |
| 34 | Workflow | 승인 처리 이력 | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, actor_id, signature_id` | Critical | Y |
| 35 | Traceability | 공통 추적 관계 | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 36 | File | 증적 파일 연결 | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 37 | Validation | 프로젝트 참여자 | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 38 | Validation | 밸리데이션 활동 마스터 | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 39 | Validation | 프로젝트 수행 활동 | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 40 | Validation | 활동 선후행 조건 | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 41 | DDS | 상세 설계 명세서 | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by` | High | Y |
| 42 | DDS | DDS 상세 항목 | [`dds_item`](#table-dds_item) | `dds_item_id` | `dds_id, created_by, updated_by` | High | Y |
| 43 | Deviation | 일탈 관리 | [`deviation`](#table-deviation) | `deviation_id` | `project_id, resolved_by, approved_by, created_by, updated_by` | Critical | Y |
| 44 | Report | 리포트 생성 작업 | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` | High | Y |
| 45 | AI | AI 생성 작업 | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 46 | AI | AI 생성 결과 | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, created_by, updated_by` | High | Y |
| 47 | AI | AI 생성 결과 항목 | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 48 | Notification | 알림 발송 | [`notification_delivery`](#table-notification_delivery) | `notification_delivery_id` | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` | High | Y |
| 49 | System | 백업 실행 이력 | [`backup_execution`](#table-backup_execution) | `backup_execution_id` | `requested_by, created_by, updated_by` | Critical | Y |
| 50 | Report | 리포트 실행 일정 | [`report_schedule`](#table-report_schedule) | `report_schedule_id` | `project_id, created_by, updated_by` | High | Y |
| 51 | File | 파일 정리 실행 이력 | [`file_cleanup_execution`](#table-file_cleanup_execution) | `file_cleanup_execution_id` | `requested_by, created_by, updated_by` | High | Y |

## 4. 도메인별 바로가기

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`evidence_link`](#table-evidence_link), [`file_cleanup_execution`](#table-file_cleanup_execution)
- **System**: [`system_asset`](#table-system_asset), [`backup_execution`](#table-backup_execution)
- **Library**: [`library_item`](#table-library_item)
- **Validation**: [`validation_project`](#table-validation_project), [`project_member`](#table-project_member), [`validation_activity`](#table-validation_activity), [`project_activity`](#table-project_activity), [`activity_dependency`](#table-activity_dependency)
- **QIA**: [`qia_assessment`](#table-qia_assessment), [`qia_module_item`](#table-qia_module_item)
- **VA**: [`vendor_audit`](#table-vendor_audit)
- **URS**: [`requirement`](#table-requirement)
- **FDS**: [`fds_spec`](#table-fds_spec), [`fds_item`](#table-fds_item), [`fds_interface`](#table-fds_interface)
- **DQ**: [`dq_assessment`](#table-dq_assessment), [`dq_item`](#table-dq_item)
- **FRA**: [`fra_assessment`](#table-fra_assessment), [`fra_item`](#table-fra_item)
- **IQ**: [`iq_assessment`](#table-iq_assessment), [`iq_item`](#table-iq_item)
- **OQ**: [`oq_assessment`](#table-oq_assessment), [`oq_item`](#table-oq_item)
- **PQ**: [`pq_assessment`](#table-pq_assessment), [`pq_item`](#table-pq_item)
- **RTM**: [`rtm_assessment`](#table-rtm_assessment), [`rtm_item`](#table-rtm_item)
- **VSR**: [`vsr_assessment`](#table-vsr_assessment), [`vsr_item`](#table-vsr_item)
- **Workflow**: [`workflow_instance`](#table-workflow_instance), [`workflow_step`](#table-workflow_step), [`approval_action`](#table-approval_action)
- **Traceability**: [`traceability_link`](#table-traceability_link)
- **DDS**: [`dds_spec`](#table-dds_spec), [`dds_item`](#table-dds_item)
- **Deviation**: [`deviation`](#table-deviation)
- **Report**: [`report_generation`](#table-report_generation), [`report_schedule`](#table-report_schedule)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item)
- **Notification**: [`notification_delivery`](#table-notification_delivery)

---

# 5. 상세 테이블 및 컬럼 정의

## Organization

<a id="table-organization"></a>
### 1. 조직/고객사 (`organization`)

| 항목 | 정의 |
|---|---|
| 설명 | 고객사 또는 운영 조직 기본정보 |
| Primary Key | `organization_id` |
| 주요 참조(FK) | - |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 14 | 조직 ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 조직 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 15 | 조직 코드 | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | 조직/회사 식별 코드 | `ORG-SAMPLE-01` |
| 16 | 조직명 | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 회사/사업장명 | `샘플 주식회사` |
| 17 | 조직 유형 | `organization_type` | `varchar(50)` | N | N | - | Y | `본사'` | N | N | N | Y | 본사 \| 공장 \| 연구소 \| 해외법인 | `본사` |
| 18 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 19 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 조직 설명 | `기업 IT 및 데이터 운영 조직` |
| 20 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00` |
| 21 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Security

<a id="table-app_user"></a>
### 2. 사용자 (`app_user`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자 계정 및 기본 프로필 정보 |
| Primary Key | `user_id` |
| 주요 참조(FK) | `organization_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1 | 사용자 ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 계정 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 2 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | organization.organization_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 3 | 로그인 계정명 | `username` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | 로그인 ID (중복 불가) | `user.sample` |
| 4 | 비밀번호 해시 | `password_hash` | `varchar(255)` | N | N | - | Y | - | N | N | N | N | 비밀번호 해시값 | `[REDACTED_PASSWORD_HASH]` |
| 5 | 사용자 실명 | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 사용자 이름 | `홍길동` |
| 6 | 이메일 | `email` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | 이메일 주소 | `user.sample@example.com` |
| 7 | 부서명 | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 소속 부서명 | `정보전략팀` |
| 8 | 직급/직책 | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 직급 정보 | `선임` |
| 9 | 계정 상태 | `status` | `varchar(20)` | N | N | - | Y | `ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 10 | 최종 로그인 시각 | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 시스템 접속 타임스탬프 | `2026-08-26T16:00:00` |
| 11 | 비밀번호 변경일 | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 비밀번호 마지막 변경 시각 | `2026-08-01T09:00:00` |
| 12 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00` |
| 13 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-role"></a>
### 3. 역할 (`role`)

| 항목 | 정의 |
|---|---|
| 설명 | 작성자·검토자·승인자 등 권한 역할 마스터 |
| Primary Key | `role_id` |
| 주요 참조(FK) | - |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 22 | 역할 ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 역할/권한 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 23 | 역할 코드 | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | 역할 식별 코드 | `SYSTEM_ADMIN` |
| 24 | 역할명 | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 역할 명칭 (시스템관리자, 작성자, 승인자 등) | `전체관리자` |
| 25 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 역할 상세 권한 범위 설명 | `시스템 전체 관리 권한` |
| 26 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-user_role"></a>
### 4. 사용자 역할 (`user_role`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자-역할 매핑 정보 |
| Primary Key | `user_role_id` |
| 주요 참조(FK) | `user_id, role_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 27 | 매핑 ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자-역할 매핑 고유 식별자. 활성 데이터는 (user_id, role_id) 조합의 중복 등록을 허용하지 않음 | `00000000-0000-0000-0000-000000000001` |
| 28 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 29 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | role.role_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 30 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Compliance

<a id="table-electronic_signature"></a>
### 5. 전자서명 (`electronic_signature`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서 검토/승인 시 전자서명 증적 기록 |
| Primary Key | `signature_id` |
| 주요 참조(FK) | `signer_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 31 | 전자서명 ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 전자서명 기록 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 32 | 서명자 ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 33 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 서명이 적용된 테이블명 (예: requirement) | `requirement` |
| 34 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 서명이 적용된 레코드 PK값 | `00000000-0000-0000-0000-000000000001` |
| 35 | 서명 단계 | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | REVIEW \| APPROVE \| REJECT | `APPROVE` |
| 36 | 서명 목적 | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 21 CFR Part 11 서명 사유 | `URS 요구사항 최종 승인` |
| 37 | 서명 타임스탬프 | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 전자서명 수행 타임스탬프 | `2026-08-26T16:30:00` |
| 38 | 대상 문서 버전 | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 전자서명이 적용된 대상 문서 Revision의 표시 버전 | `v1.0` |
| 39 | 서명 대상 내용 해시 | `content_hash` | `varchar(128)` | N | N | - | Y | - | N | N | N | Y | 서명 당시 대상 문서 및 상세 내용의 무결성 검증을 위한 SHA-256 해시값 | `[SAMPLE_SHA256_HASH]` |
| 40 | 재인증 방식 | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 전자서명 수행 시 서명자 본인 확인에 사용한 재인증 방식 | `PASSWORD` |
| 41 | 재인증 결과 | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 전자서명 수행 시 재인증 처리 결과 | `SUCCESS` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-audit_trail"></a>
### 6. Audit Trail (`audit_trail`)

| 항목 | 정의 |
|---|---|
| 설명 | 주요 데이터 변경 전후 및 수행자 감사 추적 기록 |
| Primary Key | `audit_id` |
| 주요 참조(FK) | `actor_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 42 | 감사추적 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 감사추적 레코드 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 43 | 수행자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 (시스템 자동 시 NULL 가능) | `00000000-0000-0000-0000-000000000001` |
| 44 | 작업 유형 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL | `REPORT_GENERATE` |
| 45 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 변경이 발생한 물리 테이블명 | `requirement` |
| 46 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 변경 대상 레코드 PK값 | `00000000-0000-0000-0000-000000000001` |
| 47 | 변경 전 데이터 | `old_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | 수정 전 JSON 데이터 | `{"status": "작성중"}` |
| 48 | 변경 후 데이터 | `new_values` | `jsonb` | N | N | - | N | - | N | N | N | Y | 수정 후 JSON 데이터 | `{"status": "승인완료"}` |
| 49 | 변경 사유 | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | 21 CFR Part 11 데이터 변경 사유 | `요구사항 오탈자 수정 및 규정 항목 보완` |
| 50 | 접속 IP 주소 | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | N | Y | 사용자 클라이언트 IP | `192.0.2.10` |
| 51 | 발생 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 감사추적 로그 생성 시각 (UTC) | `2026-08-26T16:30:00` |
| 52 | 수행 주체 유형 | `actor_type` | `varchar(20)` | N | N | - | Y | `USER` | N | Y | N | Y | 변경 수행 주체 유형. USER, SYSTEM, BATCH로 구분하며 USER인 경우 actor_id를 필수로 저장 | `USER` |
| 53 | 요청 ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 하나의 화면·API 요청에서 발생한 여러 감사추적 기록을 동일 요청으로 묶기 위한 식별자 | `00000000-0000-0000-0000-000000000001` |
| 54 | 세션 ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 변경 작업이 발생한 사용자 로그인 세션 식별자. 시스템·배치 처리 또는 세션이 없는 요청은 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 55 | 대상 문서 버전 | `target_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 표시 버전을 저장하며 일반 테이블은 NULL 허용 | `v1.0` |
| 56 | 대상 개정 순번 | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 숫자형 개정 순번을 저장하며 일반 테이블은 NULL 허용 | `1` |
| 57 | 요청 경로 | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 변경을 발생시킨 화면 또는 API 요청 경로. 시스템·배치 처리 등 경로가 없는 경우 NULL 허용 | `/api/fds/approve` |
| 58 | 접속 클라이언트 정보 | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | 변경 요청에 사용된 브라우저, 운영체제 또는 클라이언트 애플리케이션 정보 | `Sample-Client/1.0` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## File

<a id="table-file_asset"></a>
### 7. 파일 자산 (`file_asset`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자 첨부파일, 증적파일, 생성 리포트 및 데이터 내보내기 파일의 저장 메타데이터 관리 |
| Primary Key | `file_id` |
| 주요 참조(FK) | `uploader_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 파일 유형 및 연결 대상의 보존정책에 따라 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 59 | 파일 ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 첨부파일 메타데이터 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 60 | 업로드자 ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 업로드 파일은 업로드자 ID 필수. 시스템·정기 배치 생성 파일은 NULL 허용 | `UUID` |
| 61 | 원본 파일명 | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 업로드 당시 파일명 | `sample_document.pdf` |
| 62 | 저장 파일경로 | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | 스토리지 저장 경로/S3 Key | `documents/2026/08/sample_001.pdf` |
| 63 | 파일 구분 | `file_category` | `varchar(30)` | N | N | - | Y | `ATTACHMENT` | N | Y | N | Y | 파일 업무 구분. ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 64 | 파일 용량 | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | 파일 크기 (Byte) | `1048576` |
| 65 | MIME 타입 | `mime_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 파일 형식 | `application/pdf` |
| 66 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T16:30:00` |
| 67 | 임시파일 여부 | `is_temporary` | `boolean` | N | N | - | Y | `False` | N | Y | N | Y | 업로드 또는 생성 과정에서 발생한 임시파일 여부 | `True` |
| 68 | 만료 시각 | `expires_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 파일 정리 가능 시각. 영구 또는 업무 보존 대상 파일은 NULL | `2026-09-09 18:00:00+00` |
| 69 | 정리 상태 | `cleanup_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | 파일 정리 상태. ACTIVE, CLEANUP_PENDING, CLEANED, CLEANUP_FAILED | `ACTIVE` |
| 70 | 마지막 정리 실행 ID | `cleanup_execution_id` | `uuid` | N | Y | `file_cleanup_execution.file_cleanup_execution_id` | N | - | N | Y | N | Y | 해당 파일을 마지막으로 처리한 파일 정리 작업 ID | `UUID` |
| 71 | 정리 완료 시각 | `cleaned_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 실제 스토리지 파일 및 메타데이터 정리가 완료된 시각 | `2026-09-09 19:00:00+00` |
| 72 | 정리 실패 사유 | `cleanup_error_message` | `text` | N | N | - | N | - | N | N | N | Y | 개별 파일 정리 실패 상세 내용. 접근키 등 민감정보 저장 금지 | `파일에 대한 접근 권한 없음` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-evidence_link"></a>
### 36. 증적 파일 연결 (`evidence_link`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서·시험 항목과 증적 파일 간 N:M 연결 관리 |
| Primary Key | `evidence_link_id` |
| 주요 참조(FK) | `project_id, file_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 523 | 증적 연결 ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서·시험 항목과 증적 파일 간 연결 고유 식별자. 활성 데이터는 (project_id, file_id, target_entity_type, target_entity_id, evidence_type) 조합의 중복 등록을 허용하지 않음 | `UUID` |
| 524 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 증적 연결이 속한 프로젝트 ID | `UUID` |
| 525 | 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | 연결되는 증적 파일 ID | `UUID` |
| 526 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 증적 파일 연결 대상 유형. REQUIREMENT, FDS_SPEC, FDS_ITEM, FDS_INTERFACE, DDS_SPEC, DDS_ITEM, QIA_ASSESSMENT, QIA_MODULE_ITEM, VENDOR_AUDIT, DQ_ASSESSMENT, DQ_ITEM, FRA_ASSESSMENT, FRA_ITEM, IQ_ASSESSMENT, IQ_ITEM, OQ_ASSESSMENT, OQ_ITEM, PQ_ASSESSMENT, PQ_ITEM, RTM_ASSESSMENT, RTM_ITEM, VSR_ASSESSMENT, VSR_ITEM, DEVIATION | `IQ_ITEM` |
| 527 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | target_entity_type에 해당하는 테이블의 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `UUID` |
| 528 | 증적 유형 | `evidence_type` | `varchar(50)` | N | N | - | Y | `TEST_RESULT` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 529 | 증적 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 증적 파일의 내용 및 연결 목적 | `IQ 수행 결과 화면 캡처` |
| 530 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 531 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 생성한 사용자 ID | `UUID` |
| 532 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 533 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 최종 수정한 사용자 ID | `UUID` |
| 534 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 증적 연결 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-file_cleanup_execution"></a>
### 51. 파일 정리 실행 이력 (`file_cleanup_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | 임시파일 및 만료파일 정리 작업의 실행 조건, 처리 건수, 실행 상태, 실패 및 재시도 이력 관리 |
| Primary Key | `file_cleanup_execution_id` |
| 주요 참조(FK) | `requested_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 파일 정리 결과와 실행이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 762 | 파일 정리 실행 ID | `file_cleanup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 임시파일·만료파일 정리 작업 고유 식별자 | `UUID` |
| 763 | 정리 유형 | `cleanup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | 정리 유형. TEMPORARY_FILE, EXPIRED_FILE, ORPHAN_FILE | `EXPIRED_FILE` |
| 764 | 실행 방식 | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | 실행 방식. SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 765 | 대상 기준시각 | `target_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 해당 시각 이전에 만료되거나 정리 대상이 된 파일을 조회하는 기준시각 | `2026-09-02 00:00:00+00` |
| 766 | 실행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 실행 상태. PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 767 | 조회 파일 건수 | `scanned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 정리 대상 판정을 위해 조회한 파일 건수. 0 이상 | `100` |
| 768 | 정리 대상 건수 | `target_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 정리 대상으로 판정된 파일 건수. 0 이상 | `20` |
| 769 | 정리 완료 건수 | `cleaned_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 실제 파일과 메타데이터 정리가 완료된 건수. 0 이상 | `19` |
| 770 | 정리 실패 건수 | `failed_file_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 정리 처리에 실패한 파일 건수. 0 이상 | `1` |
| 771 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수동 정리를 요청한 사용자. 정기 배치 실행은 NULL 허용 | `UUID` |
| 772 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 수동 요청이 접수되거나 정기 작업이 등록된 시각 | `2026-09-02 01:00:00+00` |
| 773 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 파일 정리 작업이 실제 시작된 시각 | `2026-09-02 01:00:05+00` |
| 774 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 정리 작업이 성공 또는 최종 실패로 종료된 시각 | `2026-09-02 01:05:00+00` |
| 775 | 재시도 횟수 | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 최초 실행 실패 후 수행한 재시도 횟수. 0 이상 | `0` |
| 776 | 최대 재시도 횟수 | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | 자동 재시도 최대 허용 횟수. 0 이상 | `3` |
| 777 | 다음 재시도 시각 | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | RETRY_WAIT 상태 작업의 다음 실행 예정 시각 | `2026-09-02 01:15:00+00` |
| 778 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 파일 정리 실패 원인을 분류하는 시스템 오류 코드 | `FILE_DELETE_FAILED` |
| 779 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 파일 정리 실패 상세 내용. 접근키 등 민감정보 저장 금지 | `스토리지 파일 삭제 실패` |
| 780 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 파일 정리 실행이력 생성 시각(UTC) | `2026-09-02 01:00:00+00` |
| 781 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 생성 시 사용자 ID를 저장하며 시스템·배치 생성 시 NULL 허용 | `UUID` |
| 782 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 파일 정리 실행이력 최종 수정 시각(UTC) | `2026-09-02 01:05:00+00` |
| 783 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 수정 시 사용자 ID를 저장하며 시스템·배치 처리 시 NULL 허용 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## System

<a id="table-system_asset"></a>
### 8. 시스템/장비 식별 정보 (`system_asset`)

| 항목 | 정의 |
|---|---|
| 설명 | 밸리데이션 대상 시스템/장비 기준 정보 (관리번호, GAMP 범주) |
| Primary Key | `system_id` |
| 주요 참조(FK) | `organization_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 73 | 시스템 ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 시스템 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 74 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | 시스템/장비가 소속된 고객사 또는 운영 조직 | `00000000-0000-0000-0000-000000000001` |
| 75 | 관리번호 | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 자산/설비 관리번호 | `EQ-MES-2024-001` |
| 76 | 시스템명 | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 시스템/장비명 | `Sample System` |
| 77 | 시스템 유형 | `system_type` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IT시스템 \| 생산 장비 \| 품질 장비 \| 유틸리티 | `IT시스템` |
| 78 | 담당부서 | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | N | 관리/운용 담당부서 | `생산기술팀` |
| 79 | 설치위치 | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | N | 물리적/논리적 설치 장소 | `서버실 A동 3F` |
| 80 | 공급업체 | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | N | 장비/시스템 공급업체명 | `Sample Vendor` |
| 81 | 모델명 | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | N | 장비/시스템 모델명 | `FillMaster 500` |
| 82 | 시스템 설명 | `description` | `text` | N | N | - | N | - | N | N | N | N | 시스템 목적 및 운영 범위 설명 | `생산 공정 데이터 수집 및 제어` |
| 83 | 시스템 식별 상태 | `identification_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 대상 시스템 등록·식별 상태. PENDING, COMPLETED | `COMPLETED` |
| 84 | CS 포함 여부 | `is_cs_included` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | Computerized System 포함 여부 | `True` |
| 85 | 버전 | `version` | `varchar(50)` | N | N | - | N | - | N | N | N | N | 소프트웨어 또는 설비 버전 | `v3.2.1` |
| 86 | GAMP 범주 | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | N | Category 3 \| Category 4 \| Category 5 등 | `Category 4` |
| 87 | GxP 구분 | `gxp_type` | `varchar(50)` | N | N | - | N | - | N | N | N | N | GMP \| GLP \| GDP \| Non-GxP 등 | `GMP` |
| 88 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `활성'` | N | N | N | Y | 활성 \| 검토중 \| 비활성 | `활성` |
| 89 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00` |
| 90 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-backup_execution"></a>
### 49. 백업 실행 이력 (`backup_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | 시스템 데이터 및 파일의 정기·수동 백업 실행 상태, 백업 범위, 저장 위치, 실패 및 재시도 이력 관리 |
| Primary Key | `backup_execution_id` |
| 주요 참조(FK) | `requested_by, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 백업 정책 및 규정에 따라 백업 실행이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 723 | 백업 실행 ID | `backup_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 백업 실행이력 고유 식별자 | `UUID` |
| 724 | 백업 유형 | `backup_type` | `varchar(30)` | N | N | - | Y | - | N | Y | N | Y | 백업 유형. FULL, INCREMENTAL, DATABASE, FILE | `FULL` |
| 725 | 실행 방식 | `execution_type` | `varchar(20)` | N | N | - | Y | `SCHEDULED` | N | Y | N | Y | 실행 방식. SCHEDULED, ON_DEMAND | `SCHEDULED` |
| 726 | 백업 대상 | `backup_target` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 백업 대상 구분. DATABASE, FILE_STORAGE, ALL | `ALL` |
| 727 | 백업 기준시각 | `backup_base_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 백업 대상 데이터의 기준시각 | `2026-09-02 18:00:00+00` |
| 728 | 실행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 실행 상태. PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `COMPLETED` |
| 729 | 백업 저장 위치 | `backup_location` | `text` | N | N | - | N | - | N | N | N | Y | 백업 파일 저장 위치 또는 스토리지 경로. 접근 토큰 등 인증정보 저장 금지 | `backups/YYYY/MM/DD/full` |
| 730 | 백업 파일 크기 | `backup_size_bytes` | `bigint` | N | N | - | N | - | N | N | N | Y | 생성된 전체 백업 파일 크기(Byte) | `1073741824` |
| 731 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수동 실행 요청 사용자. 정기 배치 실행은 NULL 허용 | `UUID` |
| 732 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 수동 요청이 접수되거나 정기 백업이 등록된 시각 | `2026-09-02 18:00:00+00` |
| 733 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 백업 작업이 실제 시작된 시각 | `2026-09-02 18:00:05+00` |
| 734 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 백업 성공 또는 최종 실패로 작업이 종료된 시각 | `2026-09-02 18:20:00+00` |
| 735 | 재시도 횟수 | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 최초 실행 실패 후 수행한 재시도 횟수. 0 이상 | `0` |
| 736 | 최대 재시도 횟수 | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | 자동 재시도 최대 허용 횟수. 0 이상 | `3` |
| 737 | 다음 재시도 시각 | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | RETRY_WAIT 상태 작업의 다음 실행 예정 시각 | `2026-09-02 18:30:00+00` |
| 738 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 백업 실패 원인을 분류하는 시스템 오류 코드 | `BACKUP_STORAGE_UNAVAILABLE` |
| 739 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 백업 실패 상세 내용. 비밀번호와 접근키 등 민감정보 저장 금지 | `백업 저장소 연결 실패` |
| 740 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 백업 실행이력 생성 시각(UTC) | `2026-09-02 18:00:00+00` |
| 741 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 생성 시 사용자 ID를 저장하며 시스템·배치 생성 시 NULL 허용 | `UUID` |
| 742 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 백업 실행이력 최종 수정 시각(UTC) | `2026-09-02 18:20:00+00` |
| 743 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 수정 시 사용자 ID를 저장하며 시스템·배치 처리 시 NULL 허용 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Library

<a id="table-library_item"></a>
### 9. 라이브러리 항목 마스터 (`library_item`)

| 항목 | 정의 |
|---|---|
| 설명 | URS, IQ, OQ 재사용 가능 표준 라이브러리 항목 마스터 |
| Primary Key | `library_id` |
| 주요 참조(FK) | - |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 91 | 라이브러리 ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 라이브러리 항목 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 92 | 모듈 구분 | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | URS \| IQ \| OQ 구분 | `URS` |
| 93 | 코드 | `code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | 항목 코드 (예: URS-AT-L01) | `URS-AT-L01` |
| 94 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사추적, 교정/검증, 보안 등 | `감사추적` |
| 95 | 항목명 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 라이브러리 항목명 / 개요 | `데이터 변경 감사추적 자동 생성` |
| 96 | 요구사항 / 절차 | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | 상세 요구사항 명세 또는 실행 절차 | `모든 데이터 생성·수정·삭제 시...` |
| 97 | 기대 결과 | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ 테스트 시 기대 결과 (URS는 미사용) | - |
| 98 | 수용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 합격/불합격 판정 기준 | `데이터 변경 시 Audit Trail 자동 생성...` |
| 99 | 근거 규정 | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 관련 규정 (예: 21 CFR 11.10(e), KGMP) | `21 CFR 11.10(e)` |
| 100 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `True` | N | N | N | Y | 활성 여부 (TRUE/FALSE) | `True` |
| 101 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00` |
| 102 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Validation

<a id="table-validation_project"></a>
### 10. Validation 프로젝트 (`validation_project`)

| 항목 | 정의 |
|---|---|
| 설명 | 시스템별 밸리데이션 수행 단위 및 범위 (VP) |
| Primary Key | `project_id` |
| 주요 참조(FK) | `system_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 103 | 프로젝트 ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 104 | 프로젝트 코드 | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | 프로젝트 식별 코드 | `VP-SYS-008-20260422` |
| 105 | 프로젝트명 | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트명 | `테스트 장비3 CSV 프로젝트` |
| 106 | 시스템 ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | system_asset.system_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 107 | 진행률 | `progress_rate` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 프로젝트 진행률 (%) | `0` |
| 108 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `진행 중'` | N | N | N | Y | 진행 중 \| 완료 \| 보류 | `진행 중` |
| 109 | 시작일 | `start_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | 프로젝트 시작 일자 | `2026-04-15T00:00:00` |
| 110 | 검증 방식 | `validation_type` | `varchar(50)` | N | N | - | Y | `신규 검증'` | N | N | N | Y | 신규 검증 \| 변경 검증 \| 재검증 | `신규 검증` |
| 111 | 밸리데이션 레벨 | `validation_level` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | Level 1 \| Level 2 \| Level 3 \| Level 4 | `Level 4` |
| 112 | 프로젝트 컨텍스트 상태 | `context_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | 프로젝트 범위와 컨텍스트 확정 상태. DRAFT, CONFIRMED | `CONFIRMED` |
| 113 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 추가 메모 사항 | - |
| 114 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00` |
| 115 | GAMP 카테고리 | `gamp_category` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | GAMP 5 분류 (Category 3, 4, 5) | `Category 3` |
| 116 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 프로젝트 생성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 117 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 118 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 프로젝트 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 119 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-project_member"></a>
### 37. 프로젝트 참여자 (`project_member`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 참여 사용자와 수행 역할 관리 |
| Primary Key | `project_member_id` |
| 주요 참조(FK) | `project_id, user_id, role_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 535 | 프로젝트 참여자 ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 참여자 역할 매핑 고유 식별자. 활성 데이터는 (project_id, user_id, role_id) 조합의 중복 등록을 허용하지 않음 | `UUID` |
| 536 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 참여자가 소속된 Validation 프로젝트 ID | `UUID` |
| 537 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트에 참여하는 사용자 ID | `UUID` |
| 538 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | 프로젝트 내에서 사용자가 수행하는 역할 ID | `UUID` |
| 539 | 참여 상태 | `member_status` | `varchar(20)` | N | N | - | Y | `ACTIVE` | N | Y | N | Y | 프로젝트 참여 상태. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 540 | 참여 시작 시각 | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여가 시작된 시각 | `2026-09-01T10:00:00` |
| 541 | 참여 종료 시각 | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여가 종료된 시각. 현재 참여 중이면 NULL | - |
| 542 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 543 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 등록한 사용자 ID | `UUID` |
| 544 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 545 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 최종 수정한 사용자 ID | `UUID` |
| 546 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여 정보 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-validation_activity"></a>
### 38. 밸리데이션 활동 마스터 (`validation_activity`)

| 항목 | 정의 |
|---|---|
| 설명 | VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR 활동 기준정보 관리 |
| Primary Key | `activity_id` |
| 주요 참조(FK) | `created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 547 | 활동 ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 밸리데이션 활동 고유 식별자 | `UUID` |
| 548 | 활동 코드 | `activity_code` | `varchar(20)` | N | N | - | Y | - | Y | Y | N | Y | 활동 식별 코드. SYSTEM_IDENTIFICATION, VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR | `URS` |
| 549 | 활동명 | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 화면 표시용 활동명 | `사용자 요구사항 명세` |
| 550 | 활동 순서 | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 화면 및 업무 흐름의 기본 표시 순서 | `5` |
| 551 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 활동 마스터 사용 여부 | `True` |
| 552 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 생성 시각(UTC) | `09/01/2026 10:00:00` |
| 553 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 등록한 사용자 | `UUID` |
| 554 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 최종 수정 시각(UTC) | `09/01/2026 10:00:00` |
| 555 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 최종 수정한 사용자 | `UUID` |
| 556 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 마스터 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-project_activity"></a>
### 39. 프로젝트 수행 활동 (`project_activity`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 수행 대상 활동, 활성화 및 진행 상태 관리 |
| Primary Key | `project_activity_id` |
| 주요 참조(FK) | `project_id, activity_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 557 | 프로젝트 활동 ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 수행 활동 고유 식별자. 활성 데이터는 (project_id, activity_id) 조합의 중복을 허용하지 않음 | `UUID` |
| 558 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 활동이 속한 Validation 프로젝트 | `UUID` |
| 559 | 활동 ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 프로젝트에서 수행할 활동 | `UUID` |
| 560 | 수행 대상 여부 | `is_selected` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | VP 수행 활동에 포함된 활동인지 여부 | `True` |
| 561 | 필수 활동 여부 | `is_required` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | 해당 프로젝트에서 생략할 수 없는 활동인지 여부 | `True` |
| 562 | 활동 상태 | `activity_status` | `varchar(20)` | N | N | - | Y | `LOCKED` | N | Y | N | Y | 활동 상태. LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED | `READY` |
| 563 | 활성화 시각 | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 선행 조건 충족으로 활동이 READY가 된 시각 | `2026-09-01T10:00:00` |
| 564 | 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 시작 시각 | `2026-09-01T11:00:00` |
| 565 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 완료 시각 | `2026-09-02T15:00:00` |
| 566 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동의 최종 승인 완료 시각 | `2026-09-02T17:00:00` |
| 567 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 568 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 등록 사용자 | `UUID` |
| 569 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 570 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 최종 수정 사용자 | `UUID` |
| 571 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 활동 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-activity_dependency"></a>
### 40. 활동 선후행 조건 (`activity_dependency`)

| 항목 | 정의 |
|---|---|
| 설명 | 후행 활동의 활성화를 위한 선행 활동, 관계 구분 및 판정 조건 관리 |
| Primary Key | `activity_dependency_id` |
| 주요 참조(FK) | `successor_activity_id, predecessor_activity_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 572 | 활동 선후행 조건 ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 활동 선후행 조건 고유 식별자 | `UUID` |
| 573 | 후행 활동 ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 조건 충족 후 활성화되는 활동 | `UUID` |
| 574 | 선행 활동 ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | 후행 활동 활성화 전에 확인할 활동. 전체 활동 조건이면 NULL 허용 | `UUID` |
| 575 | 관계 구분 | `dependency_type` | `varchar(20)` | N | N | - | Y | `REQUIRED` | N | Y | N | Y | 선후행 관계 구분. REQUIRED, RECOMMENDED | `REQUIRED` |
| 576 | 요구 상태 | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | 선행 활동에 요구되는 상태. CREATED, COMPLETED, APPROVED | `APPROVED` |
| 577 | 조건 유형 | `condition_type` | `varchar(50)` | N | N | - | Y | `STATUS` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, CONTEXT_CONFIRMED, GXP_SCOPE_CONFIRMED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED | `STATUS` |
| 578 | 조건 값 | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 조건 판정에 필요한 추가 값 또는 대상 엔터티 유형 | `APPROVED` |
| 579 | 조건 설명 | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | 사람이 확인할 수 있는 활성화 조건 설명 | `URS 승인완료` |
| 580 | 평가 순서 | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 동일 후행 활동의 조건 평가 순서 | `1` |
| 581 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 활성화 조건 사용 여부 | `True` |
| 582 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 583 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 등록 사용자 | `UUID` |
| 584 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 585 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 최종 수정 사용자 | `UUID` |
| 586 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 조건 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## QIA

<a id="table-qia_assessment"></a>
### 11. 품질 영향 평가 헤더 (`qia_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 QIA 종합 평가 및 21 CFR Part 11 평가 결과 |
| Primary Key | `qia_id` |
| 주요 참조(FK) | `project_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 120 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 평가 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 121 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 122 | Part11 Q1 전자기록 생성여부 | `p11_q1` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q1 전자기록 생성/수정/유지 여부 (Yes/No) | `Yes` |
| 123 | Part11 Q2 전자저장 여부 | `p11_q2` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q2 전자형태 저장 여부 (Yes/No) | `Yes` |
| 124 | Part11 Q3 규제기관 제출여부 | `p11_q3` | `varchar(10)` | N | N | - | Y | `No'` | N | N | N | Y | Q3 규제기관 전자기록 제출 여부 (Yes/No) | `No` |
| 125 | Part11 Q4 전자서명 사용여부 | `p11_q4` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q4 전자서명 사용 여부 (Yes/No) | `Yes` |
| 126 | Part11 Q5 수기서명 대체여부 | `p11_q5` | `varchar(10)` | N | N | - | Y | `Yes'` | N | N | N | Y | Q5 수기서명을 전자서명으로 대체 여부 (Yes/No) | `Yes` |
| 127 | Part11 Q6 시스템 유형 | `p11_q6` | `varchar(20)` | N | N | - | Y | `Closed'` | N | N | N | Y | Closed System \| Open System | `Closed` |
| 128 | Part11 평가 결론 | `part11_result` | `text` | N | N | - | Y | - | N | N | N | Y | 21 CFR Part 11 적용 요건 결론 | `21 CFR Part 11 해당 시스템 (Closed System)` |
| 129 | GxP 범위 상태 | `gxp_scope_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | QIA를 통한 GxP 범위 확정 상태. DRAFT, CONFIRMED | `CONFIRMED` |
| 130 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | QIA 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 131 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | QIA 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 132 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | QIA 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 133 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 QIA 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 134 | 작성·검토·승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | 문서 진행 상태 (DRAFT, REVIEW, APPROVED 등) | `DRAFT` |
| 135 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-qia_module_item"></a>
### 12. QIA 모듈 상세 평가 (`qia_module_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 모듈/프로세스별 GxP Q1~Q10 항목 및 평가 결과 |
| Primary Key | `qia_module_item_id` |
| 주요 참조(FK) | `qia_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 136 | QIA 모듈 항목 ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 모듈 평가 항목 고유 식별자 | `UUID` |
| 137 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | 상위 QIA 평가 식별자 | `UUID` |
| 138 | 모듈 코드 | `module_code` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 상위 모듈 코드 | `QM` |
| 139 | 모듈명 | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 상위 모듈명 | `품질관리` |
| 140 | 모듈 설명 | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | 모듈의 범위 및 목적 설명 | `품질관리 관련 종합 평가` |
| 141 | 프로세스 코드 | `process_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 프로세스 식별 코드 | `PRC-001` |
| 142 | 프로세스명 | `process_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 세부 프로세스명 | `작업지시` |
| 143 | Q1 GXP목적 | `q1_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 144 | Q2 생산공정 | `q2_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 145 | Q3 생산데이터 | `q3_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 146 | Q4 품질영향 | `q4_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `O` |
| 147 | Q5 보관관리 | `q5_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 148 | Q6 출하승인 | `q6_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 149 | Q7 리콜 | `q7_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 150 | Q8 규제문서 | `q8_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 151 | Q9 문서관리 | `q9_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 152 | Q10 보안 | `q10_val` | `varchar(5)` | N | N | - | Y | `X'` | N | N | N | Y | O \| X \| △ | `X` |
| 153 | 평가 결과 | `result_type` | `varchar(20)` | N | N | - | Y | `Non-GxP'` | N | N | N | Y | GxP \| Non-GxP | `GxP` |
| 154 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-26T10:00:00` |
| 155 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-26T10:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## VA

<a id="table-vendor_audit"></a>
### 13. 공급업체 감사 평가 (`vendor_audit`)

| 항목 | 정의 |
|---|---|
| 설명 | 공급업체 점검/감사(Audit) 계획, 실행 및 결함 수치 기록 |
| Primary Key | `audit_id` |
| 주요 참조(FK) | `project_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 156 | 감사 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 공급업체 감사 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 157 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 158 | 문서 번호 | `document_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사 보고서 문서 번호 | `VA-2026-001` |
| 159 | 공급업체명 | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사 대상 공급업체명 | `Sample Vendor` |
| 160 | 대상 시스템 | `system_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사 대상 시스템명/버전 | `Sample System v1.0` |
| 161 | 감사 방식 | `audit_type` | `varchar(50)` | N | N | - | Y | `현장 감사'` | N | N | N | Y | 현장 감사 \| 서류 감사 \| 원격 감사 | `현장 감사` |
| 162 | 감사 일자 | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | 감사 수행 일자 (또는 예정일) | `2026-08-26T00:00:00` |
| 163 | 감사자 | `auditor_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 감사 담당자/수행자 이름 | `홍길동` |
| 164 | 감사 결과 | `audit_result` | `varchar(20)` | N | N | - | Y | `적합'` | N | N | N | Y | 적합 \| 조건부 적합 \| 부적합 | `적합` |
| 165 | Critical 결함 수 | `critical_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 치명적 결함(Critical) 발견 건수 | `0` |
| 166 | Major 결함 수 | `major_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 중대 결함(Major) 발견 건수 | `0` |
| 167 | Minor 결함 수 | `minor_defects` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 경미 결함(Minor) 발견 건수 | `1` |
| 168 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 감사 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 169 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 감사 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 170 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 감사 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 171 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 감사 문서 버전 여부 (TRUE/FALSE) | `True` |
| 172 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중'` | N | N | N | Y | 작성 중 \| 완료 \| 승인완료 | `작성 중` |
| 173 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 174 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## URS

<a id="table-requirement"></a>
### 14. 사용자 요구사항 명세 (`requirement`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 URS 요구사항 항목 및 문서 개정 버전 관리 |
| Primary Key | `requirement_id` |
| 주요 참조(FK) | `project_id, created_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 175 | 요구사항 ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 요구사항 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 176 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 177 | 항목 번호 | `item_number` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 요구사항 관리 번호 (예: URS-001) | `URS-001` |
| 178 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 시스템 관리, 감사추적, 전자서명 등 | `전자서명` |
| 179 | 항목 / 기능 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 요구사항 항목명 및 주요 기능 | `전자서명 서명자·일시·의미 기록` |
| 180 | 요구사항 상세 | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | 요구사항 상세 명세 내용 | `전자서명 시 서명자 ID, 서명 일시...` |
| 181 | 근거 규정 | `regulation` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 관련 법규 및 규정 (예: 21 CFR 11.50) | `21 CFR 11.50` |
| 182 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성중'` | N | N | N | Y | 작성중 \| 검토중 \| 승인완료 \| 반려 | `작성중` |
| 183 | 문서 버전 | `version` | `varchar(20)` | N | N | - | Y | `v1.0'` | N | N | N | Y | 요구사항 문서 개정 버전 (예: v1.0, v2.0) | `v1.0` |
| 184 | 개정 차수 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 버전 개정 순번 (1, 2, 3...) | `1` |
| 185 | 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 버전 여부 (TRUE/FALSE) | `True` |
| 186 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 버전 신규 생성/개정 시 사유 | `최초 작성` |
| 187 | RTM 연결 여부 | `is_rtm_linked` | `boolean` | N | N | - | Y | `False` | N | N | N | Y | Traceability Matrix 연결 여부 | `False` |
| 188 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 요구사항 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 189 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 190 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## FDS

<a id="table-fds_spec"></a>
### 15. 기능 설계 명세서 (`fds_spec`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 FDS 문서 헤더와 버전 및 상태 관리 |
| Primary Key | `fds_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 191 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 192 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 193 | FDS 번호 | `fds_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FDS 문서 번호 | `VP-SYS-008-20260422` |
| 194 | FDS 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 기능 설계 명세서 제목 | `테스트 장비3 CSV 프로젝트` |
| 195 | 문서 버전 | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | FDS 문서 표준 버전 | `v1.0` |
| 196 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 버전 개정 순번 (예: 1, 2, 3...) | `1` |
| 197 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 버전 신규 생성/개정 사유 | `최초 작성` |
| 198 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 버전 여부 (TRUE/FALSE) | `True` |
| 199 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중` | N | N | N | Y | 문서 상태 (작성 중, 검토 중, 승인 완료) | `작성 중` |
| 200 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 201 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | FDS 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 202 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 203 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | FDS 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 204 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-fds_item"></a>
### 16. FDS 상세 항목 (`fds_item`)

| 항목 | 정의 |
|---|---|
| 설명 | FDS 문서의 기능, 화면 및 상세 설계 항목 관리 |
| Primary Key | `fds_item_id` |
| 주요 참조(FK) | `fds_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 205 | FDS 항목 ID | `fds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FDS 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 206 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | N | N | Y | 상위 FDS 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 207 | 항목 구분 | `item_type` | `varchar(20)` | N | N | - | Y | `FUNCTION` | N | N | N | Y | 서브 탭 구분 (FUNCTION, SCREEN, INTERFACE) | `FUNCTION` |
| 208 | FDS 항목 번호 | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FDS 항목 관리 번호 | `FDS-001` |
| 209 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 기능 카테고리 | `시스템 관리` |
| 210 | 항목/기능 | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 기능명 및 주요 기능 | `전자서명` |
| 211 | 설명 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 기능 상세 설명 | `사용자 로그인 시 전자서명 검증 기능` |
| 212 | 관련 화면 | `related_screen` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 관련 화면명 | `사용자 관리 화면` |
| 213 | 개별 개정 차수 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 개정 순번 (1, 2, 3...) | `1` |
| 214 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중` | N | N | N | Y | 진행 상태 (작성 중, 검토 중, 승인 완료) | `작성 중` |
| 215 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 216 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 217 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 218 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 219 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-fds_interface"></a>
### 17. FDS 인터페이스 정의 (`fds_interface`)

| 항목 | 정의 |
|---|---|
| 설명 | FDS의 시스템 간 인터페이스, 연동 데이터 및 전송 방식 관리 |
| Primary Key | `fds_interface_id` |
| 주요 참조(FK) | `fds_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 220 | FDS 인터페이스 ID | `fds_interface_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FDS 인터페이스 레코드 고유 식별자 | `UUID` |
| 221 | FDS ID | `fds_id` | `uuid` | N | Y | `fds_spec.fds_id` | Y | - | N | Y | N | Y | 상위 FDS 문서 식별자 | `UUID` |
| 222 | 인터페이스 관리 번호 | `interface_id` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | FDS 문서 내 인터페이스 관리 번호 | `IF-001` |
| 223 | 소스 시스템 | `source_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 송신 시스템명 | `Sample System` |
| 224 | 대상 시스템 | `target_system` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 수신 시스템명 | `Target System` |
| 225 | 연동 데이터 | `interface_data` | `text` | N | N | - | Y | - | N | N | N | Y | 전송 데이터 항목 | `자재 소비량, 배치 결과` |
| 226 | 연동 주기 | `transfer_cycle` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 실시간, 주기적, 이벤트 기반 등 | `실시간` |
| 227 | 연동 방식 | `transfer_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | REST API, Message Queue 등 | `REST API` |
| 228 | FDS 연계 번호 | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연관 FDS 항목 번호의 화면 표시용 값 | `FDS-008` |
| 229 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-26T10:00:00` |
| 230 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 인터페이스 작성자 식별자 | `UUID` |
| 231 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-26T10:00:00` |
| 232 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 인터페이스 최종 수정자 식별자 | `UUID` |
| 233 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## DQ

<a id="table-dq_assessment"></a>
### 18. 설계 적격성 평가 (`dq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 설계 적격성 평가 문서 헤더 관리 |
| Primary Key | `dq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 234 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | DQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 235 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 236 | DQ 번호 | `dq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | DQ 문서 번호 (예: DQ-VP-SYS-008-20260422) | `DQ-VP-SYS-008-20260422` |
| 237 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 적격성 평가 문서 제목 | `설계 적격성 평가 (URS → FDS/DDS 매핑)` |
| 238 | 문서 버전 | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | DQ 문서 표준 버전 | `v1.0` |
| 239 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | DQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 240 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | DQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 241 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 DQ 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 242 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중` | N | N | N | Y | 문서 상태 (작성 중, 검토 중, 승인 완료) | `작성 중` |
| 243 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 244 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | DQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 245 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 246 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | DQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 247 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-dq_item"></a>
### 19. DQ 상세 평가 항목 (`dq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | URS와 FDS/DDS 설계의 적격성 평가 결과 관리 |
| Primary Key | `dq_item_id` |
| 주요 참조(FK) | `dq_id, requirement_id, reviewed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 248 | DQ 항목 ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | DQ 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 249 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | N | N | Y | 상위 DQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 250 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | 매핑 대상 URS 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 251 | URS 번호 | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 표시용 URS 번호 (예: URS-001) | `URS-001` |
| 252 | URS 요구사항 | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | URS 세부 요구사항 내용 | `사용자 로그인 및 전자서명 기능` |
| 253 | FDS 연계 | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계된 FDS 번호 (예: FDS-001) | `FDS-001` |
| 254 | FDS 기능 | `fds_feature_name` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 연계된 FDS 기능명 | `전자서명 검증` |
| 255 | DDS 연계 | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계된 DDS 설계 번호 | `DDS-001` |
| 256 | DDS 설계 | `dds_description` | `text` | N | N | - | N | - | N | N | N | Y | 연계된 DDS 데이터베이스/컴포넌트 설계 | `User Auth Table Schema` |
| 257 | 판정 | `result_status` | `varchar(20)` | N | N | - | Y | `검토 대기` | N | N | N | Y | 평가 판정 (PASS, FAIL, PENDING) | `PASS` |
| 258 | 검토자 ID | `reviewed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | 항목 검토 수행자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 259 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 검토 관련 특이사항 및 비고 | `FDS 및 DDS 설계 반영 완료 확인` |
| 260 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 261 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 262 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 263 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 264 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## FRA

<a id="table-fra_assessment"></a>
### 20. 기능 위험평가 (`fra_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 기능 위험평가 문서 헤더 관리 |
| Primary Key | `fra_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 265 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FRA 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 266 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 267 | FRA 번호 | `fra_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | FRA 문서 번호 (예: FRA-VP-SYS-008-20260422) | `FRA-VP-SYS-008-20260422` |
| 268 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | FMEA 기반 기능 위험평가 문서 제목 | `FMEA 기반 기능 위험평가` |
| 269 | 문서 버전 | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | FRA 문서 표준 버전 | `v1.0` |
| 270 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | FRA 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 271 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | FRA 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 272 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 FRA 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 273 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중` | N | N | N | Y | 문서 상태 (작성 중, 검토 중, 승인 완료) | `작성 중` |
| 274 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 275 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | FRA 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 276 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 277 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | FRA 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 278 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-fra_item"></a>
### 21. FRA 위험 상세 항목 (`fra_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 기능별 위험 시나리오, 위험도 및 완화 전략 관리 |
| Primary Key | `fra_item_id` |
| 주요 참조(FK) | `fra_id, requirement_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 279 | 위험 항목 ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | FRA 위험 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 280 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | N | N | Y | 상위 FRA 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 281 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | N | N | Y | 매핑 URS 요구사항 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 282 | URS 참조번호 | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 화면 표시용 URS 참조번호 (예: URS-001) | `URS-001` |
| 283 | 기능명 | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 위험 평가 대상 기능명 | `전자서명` |
| 284 | 위험 시나리오 | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | FMEA 위험 발생 시나리오 설명 | `전자서명 시 비밀번호 검증 미수행` |
| 285 | 제품영향 (PI) | `pi_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Product Impact (H, M, L) | `H` |
| 286 | 발생가능성 (LL) | `ll_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Likelihood of Occurrence (H, M, L) | `M` |
| 287 | 탐지가능성 (DL) | `dl_score` | `varchar(10)` | N | N | - | Y | `H` | N | N | N | Y | Detectability (H, M, L) | `L` |
| 288 | 위험도 수치 (RV) | `risk_value` | `integer` | N | N | - | N | `1` | N | N | N | Y | PI, LL, DL 조합 산출 위험값 (RV) | `1` |
| 289 | 위험 등급 | `risk_level` | `varchar(20)` | N | N | - | Y | `LOW` | N | Y | N | Y | 산출된 위험도 등급. LOW, MEDIUM, HIGH | `HIGH` |
| 290 | 완화 전략 | `mitigation_strategy` | `varchar(20)` | N | N | - | Y | `Test` | N | N | N | Y | 위험 완화 전략 (Test, SOP, No Action) | `Test` |
| 291 | 연결 테스트 | `test_reference` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 검증 테스트 코드 (예: OQ-AT-01) | `OQ-AT-01` |
| 292 | 진행 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성 중` | N | N | N | Y | 위험 항목 상태 (작성 중, 검토 완료) | `작성 중` |
| 293 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 294 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 295 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 296 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 297 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## IQ

<a id="table-iq_assessment"></a>
### 22. 설치 적격성 평가 (`iq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 설치 적격성 평가 문서 헤더 관리 |
| Primary Key | `iq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 298 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | IQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 299 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 300 | IQ 번호 | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ 문서 번호 (예: IQ-VP-SYS-010-20260529) | `IQ-VP-SYS-010-20260529` |
| 301 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설치 적격성 평가 문서 제목 | `Installation Qualification` |
| 302 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 303 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 304 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | IQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 305 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 IQ 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 306 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ 프로토콜 상태. DRAFT, REVIEW, APPROVED | `APPROVED` |
| 307 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | IQ 수행 레코드 상태. 프로토콜 APPROVED 이후 DRAFT, REVIEW, APPROVED 순으로 진행 | `DRAFT` |
| 308 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 309 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 310 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 311 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 312 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-iq_item"></a>
### 23. IQ 상세 테스트 항목 (`iq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 테스트 절차, 결과, 증적 및 수행자 관리 |
| Primary Key | `iq_item_id` |
| 주요 참조(FK) | `iq_id, executed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 313 | IQ 항목 ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | IQ 세부 테스트 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 314 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | 상위 IQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 315 | 절차 번호 | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 테스트 수행 절차 번호 (예: 01, 02) | `1` |
| 316 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 테스트 항목 ID (예: IQ-NEW-01) | `IQ-NEW-01` |
| 317 | 테스트 케이스 | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 하드웨어 설치, 소프트웨어 설치 등 | `하드웨어 설치` |
| 318 | URS 연계 | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 매핑 URS 번호 (예: URS-001) | `URS-001` |
| 319 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `설치될 서버의 하드웨어 사양이 URS를 충족하는지 확인` |
| 320 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `하드웨어 사양이 URS에 명시된 요구사항과 일치해야 함` |
| 321 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `승인완료` | N | N | N | Y | 항목별 프로토콜 승인 상태 | `승인완료` |
| 322 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 테스트 수행 후 실제 결과 기록 | `IQ 테스트1 결과` |
| 323 | 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 테스트 판정 (Pass, Fail, 미실행) | `Pass` |
| 324 | 수행자 ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | 테스트 수행자 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 325 | 수행일 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 테스트 수행 완료 일시 | `2026-06-22T00:00:00` |
| 326 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `작성중` | N | N | N | Y | 항목별 레코드 승인 상태 | `승인완료` |
| 327 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 328 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 329 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 330 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 331 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## OQ

<a id="table-oq_assessment"></a>
### 24. 운전 적격성 평가 (`oq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 운전 적격성 평가 문서 헤더 관리 |
| Primary Key | `oq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 332 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | OQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 333 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 334 | OQ 번호 | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ 문서 번호 (예: OQ-VP-SYS-010-20260529) | `OQ-VP-SYS-010-20260529` |
| 335 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 운전 적격성 평가 문서 제목 | `Operational Qualification` |
| 336 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 337 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 338 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | OQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 339 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 OQ 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 340 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ 프로토콜 상태. DRAFT, REVIEW, APPROVED | `APPROVED` |
| 341 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | OQ 수행 레코드 상태. 프로토콜 APPROVED 이후 DRAFT, REVIEW, APPROVED 순으로 진행 | `DRAFT` |
| 342 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 343 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 344 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 345 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 346 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-oq_item"></a>
### 25. OQ 상세 테스트 항목 (`oq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 테스트 절차, 결과, 증적 및 수행자 관리 |
| Primary Key | `oq_item_id` |
| 주요 참조(FK) | `oq_id, executed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 347 | OQ 항목 ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | OQ 세부 테스트 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 348 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | 상위 OQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 349 | 절차 번호 | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 테스트 수행 절차 번호 (예: 01, 02, 03) | `1` |
| 350 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 테스트 항목 ID (예: OQ-AT-L01) | `OQ-AT-L01` |
| 351 | 테스트 케이스 | `test_case` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Audit Trail, 사용자 관리, 백업 및 복구 등 | `Audit Trail` |
| 352 | URS 연계 | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 매핑 URS 번호 (예: URS-001) | `URS-001` |
| 353 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `사용자 데이터 변경 시 감사추적 자동 생성` |
| 354 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `변경 전/후 값, 사용자, 날짜/시간, IP 기록` |
| 355 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `승인완료` | N | N | N | Y | 항목별 프로토콜 승인 상태 | `승인완료` |
| 356 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 테스트 수행 후 실제 결과 기록 | `실제 결과 Test` |
| 357 | 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 테스트 판정 (Pass, Fail, 미실행) | `Pass` |
| 358 | 수행자 ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | 테스트 수행자 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 359 | 수행일 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 테스트 수행 완료 일시 | `2026-06-22T00:00:00` |
| 360 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `작성중` | N | N | N | Y | 항목별 레코드 승인 상태 | `승인완료` |
| 361 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 362 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 363 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 364 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 365 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## PQ

<a id="table-pq_assessment"></a>
### 26. 성능 적격성 평가 (`pq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 성능 적격성 평가 문서 헤더 및 수행계획 관리 |
| Primary Key | `pq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 366 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | PQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 367 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 368 | PQ 번호 | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ 문서 번호 (예: PQ-VP-SYS-008-20260422) | `PQ-VP-SYS-008-20260422` |
| 369 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 성능 적격성 평가 문서 제목 | `Performance Qualification` |
| 370 | PQ 시작 예정일 | `start_scheduled_date` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | PQ 시작 예정 기준 (예: OQ 완료 + 5영업일) | `OQ 완료 + 5영업일` |
| 371 | PQ 완료 목표일 | `target_completion_date` | `date` | N | N | - | N | - | N | N | N | Y | PQ 완료 목표 일자 | `2024-04-15T00:00:00` |
| 372 | 수행 방식 | `execution_method` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | PQ 수행 방식 (예: 실제 생산 데이터 3배치 이상) | `실제 생산 데이터 3배치 이상` |
| 373 | 주요 수행자 | `primary_executor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 주요 수행 담당 팀/부서 (예: 프로세스 오너 (생산팀)) | `프로세스 오너 (생산팀)` |
| 374 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 375 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 376 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | PQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 377 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 PQ 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 378 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성중` | N | N | N | Y | PQ 평가 전체 진행 상태 (수행 대기 중, 진행 중, 완료) | `PQ 수행 대기 중` |
| 379 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ 프로토콜 상태. DRAFT, REVIEW, APPROVED | `APPROVED` |
| 380 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | PQ 수행 레코드 상태. 프로토콜 APPROVED 이후 DRAFT, REVIEW, APPROVED 순으로 진행 | `DRAFT` |
| 381 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 382 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 383 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 384 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 385 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-pq_item"></a>
### 27. PQ 상세 테스트 항목 (`pq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 테스트 절차, 결과, 증적 및 수행자 관리 |
| Primary Key | `pq_item_id` |
| 주요 참조(FK) | `pq_id, executed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 386 | PQ 항목 ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | PQ 세부 테스트 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 387 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | 상위 PQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 388 | 절차 번호 | `step_no` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 테스트 수행 절차 번호 (예: 01, 02) | `1` |
| 389 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | PQ 평가 카테고리 (예: 연속 배치 검증, 데이터 완전성 등) | `연속 배치 검증` |
| 390 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | PQ 테스트 검증 수행 상세 절차 | `연속 3배치 이상 생산 공정 정상 완료 검증` |
| 391 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `모든 배치가 사양에 맞게 정상 생산 완료되어야 함` |
| 392 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 테스트 수행 후 실제 결과 기록 | `연속 3배치 정상 완료 확인` |
| 393 | 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 테스트 판정 (Pass, Fail, 미실행) | `Pass` |
| 394 | 수행자 ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | N | N | Y | 테스트 수행자 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 395 | 수행일 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 테스트 수행 완료 일시 | `2026-08-26T00:00:00` |
| 396 | 진행 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성중` | N | N | N | Y | 항목별 진행 상태 (작성중, 승인완료) | `작성중` |
| 397 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 398 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 399 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 400 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 401 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## RTM

<a id="table-rtm_assessment"></a>
### 28. 요구사항 추적 매트릭스 (`rtm_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 요구사항 추적성과 산출물 커버리지 집계 및 승인 시점의 RTM 스냅샷 관리 |
| Primary Key | `rtm_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 402 | RTM ID | `rtm_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 403 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 404 | RTM 번호 | `rtm_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | RTM 문서 번호 (예: RTM-VP-SYS-008-20260422) | `RTM-VP-SYS-008-20260422` |
| 405 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 요구사항 추적 매트릭스 문서 제목 | `요구사항 추적 매트릭스 (자동 생성)` |
| 406 | URS 전체 건수 | `total_urs_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 프로젝트 전체 URS 요구사항 건수 | `8` |
| 407 | FRA 연계율 | `fra_link_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | FRA 리스크 평가 연계율 (%) | `88` |
| 408 | IQ 커버리지 | `iq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | IQ 테스트 커버리지 (%) | `0` |
| 409 | OQ 커버리지 | `oq_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | OQ 테스트 커버리지 (%) | `44` |
| 410 | 전체 평균 커버리지 | `avg_coverage_rate` | `numeric(5,2)` | N | N | - | N | `0` | N | N | N | Y | 전체 산출물 통합 평균 커버리지 (%) | `44` |
| 411 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | RTM 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 412 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | RTM 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 413 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | RTM 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 414 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 RTM 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 415 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `작성중` | N | N | N | Y | RTM 진행 상태 (작성중, 승인완료) | `작성중` |
| 416 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 417 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | RTM 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 418 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 419 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | RTM 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 420 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-rtm_item"></a>
### 29. RTM 상세 추적 항목 (`rtm_item`)

| 항목 | 정의 |
|---|---|
| 설명 | traceability_link를 기준으로 생성된 URS별 FRA, FDS, DDS, IQ, OQ, PQ 추적 결과 및 승인 시점의 상세 스냅샷 관리 |
| Primary Key | `rtm_item_id` |
| 주요 참조(FK) | `rtm_id, requirement_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 421 | RTM 항목 ID | `rtm_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | RTM 매트릭스 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 422 | RTM ID | `rtm_id` | `uuid` | N | Y | `rtm_assessment.rtm_id` | Y | - | N | N | N | Y | 상위 RTM 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 423 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | N | N | Y | 매핑 URS 요구사항 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 424 | URS 번호 | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 표시용 URS 번호 (예: URS-001) | `URS-001` |
| 425 | 항목/기능 | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | URS 요구사항 기능명 | `전자서명 서명자·일시·의미 기록` |
| 426 | 요구사항 내용 | `requirement_desc` | `text` | N | N | - | Y | - | N | N | N | Y | URS 세부 요구사항 내용 | `전자서명 시 서명자 ID, 서명 일시가 기록되어야 한다.` |
| 427 | 채택 여부 | `adoption_status` | `varchar(10)` | N | N | - | Y | `O` | N | N | N | Y | URS 채택 여부 (O, X) | `O` |
| 428 | FRA 매핑 | `fra_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 FRA 번호 (예: FRA-001, N/A) | `N/A` |
| 429 | FDS 매핑 | `fds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 FDS 번호 (예: FDS-001, 매핑 실패) | `매핑 실패` |
| 430 | DDS 매핑 | `dds_mapping` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 DDS 번호 (예: DDS-001, 매핑 실패) | `매핑 실패` |
| 431 | IQ 결과 | `iq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 IQ 테스트 항목 번호 (예: IQ-NEW-07) | `IQ-NEW-07` |
| 432 | OQ 결과 | `oq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 OQ 테스트 항목 번호 (예: OQ-AT-L01, 매핑 실패) | `매핑 실패` |
| 433 | PQ 결과 | `pq_result` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 연계 PQ 테스트 항목 번호 (예: PQ-01, N/A) | `N/A` |
| 434 | 항목 커버리지 | `item_coverage_rate` | `numeric(5,2)` | N | N | - | Y | `0` | N | N | N | Y | 해당 URS 항목의 산출물 커버리지 비율 (%) | `50` |
| 435 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 436 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 437 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 438 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 439 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## VSR

<a id="table-vsr_assessment"></a>
### 30. 밸리데이션 종합 보고서 (`vsr_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 최종 밸리데이션 결론과 종합 보고서 관리 |
| Primary Key | `vsr_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 440 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 441 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 442 | VSR 번호 | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR 문서 번호 (예: VSR-VP-SYS-008-20260422) | `VSR-VP-SYS-008-20260422` |
| 443 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 밸리데이션 종합 보고서 제목 | `Validation Summary Report` |
| 444 | 밸리데이션 결론 | `overall_conclusion` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 최종 적합성 결론 (적합, 조건부 적합, 부적합) | `조건부 적합 (Conditionally Acceptable)` |
| 445 | 결론 상세 설명 | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | 결론 사유 및 조건사항 (예: OQ-GMP-02 일탈 해결 완료 후 최종 승인 가능) | `OQ-GMP-02 일탈 해결 완료 후 최종 승인 가능` |
| 446 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 447 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 448 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | VSR 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 449 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 활성화된 최신 VSR 평가 문서 버전 여부 (TRUE/FALSE) | `True` |
| 450 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `검토중` | N | N | N | Y | VSR 진행 상태 (작성중, 검토중, 승인완료) | `검토중` |
| 451 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 452 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 453 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 454 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 455 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-vsr_item"></a>
### 31. VSR 활동 요약 항목 (`vsr_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 밸리데이션 활동별 문서, 결과, 일탈 및 승인 정보 요약 관리 |
| Primary Key | `vsr_item_id` |
| 주요 참조(FK) | `vsr_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 456 | VSR 항목 ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | N | N | Y | VSR 활동 요약 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 457 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | 상위 VSR 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 458 | 활동 구분 | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 밸리데이션 활동 구분 (VP, QIA, VA, URS, FDS, DQ, FRA, IQ, OQ, PQ, RTM) | `URS` |
| 459 | 문서 번호 | `doc_no` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 해당 단계 문서 번호 | `VP-SYS-008-20260422 · URS` |
| 460 | 개정 차수 | `revision_no` | `varchar(20)` | N | N | - | Y | `1` | N | N | N | Y | 문서 개정 차수 (REV) | `2.1` |
| 461 | 수행일 | `execution_date` | `date` | N | N | - | N | - | N | N | N | Y | 활동 수행 완료일자 | `2024-02-15T00:00:00` |
| 462 | 성공 건수 | `pass_count` | `integer` | N | N | - | N | - | N | N | N | Y | PASS 테스트/요구사항 건수 | `125` |
| 463 | 실패 건수 | `fail_count` | `integer` | N | N | - | N | - | N | N | N | Y | FAIL 건수 | `0` |
| 464 | 일탈 건수 | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 일탈 발생 및 해결 수량 (예: 3건(해결2), 1(미결)) | - |
| 465 | 결론/상태 | `item_status` | `varchar(20)` | N | N | - | Y | `대기` | N | N | N | Y | 해당 문서/활동 결론 (승인완료, 검토중, 대기) | `승인완료` |
| 466 | 승인자명 | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최종 승인자 성명 | `홍길동` |
| 467 | 승인일 | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | 최종 승인 일자 | `2024-02-20T00:00:00` |
| 468 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00` |
| 469 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 470 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00` |
| 471 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 472 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Workflow

<a id="table-workflow_instance"></a>
### 32. Workflow 인스턴스 (`workflow_instance`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서별 검토·승인 Workflow 진행 정보 관리 |
| Primary Key | `workflow_instance_id` |
| 주요 참조(FK) | `requested_by, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 473 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서별 검토·승인 Workflow 고유 식별자 | `UUID` |
| 474 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | Workflow 대상 문서의 물리 테이블명 | `fds_spec` |
| 475 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | Workflow 대상 문서 Revision 행의 PK값 | `UUID` |
| 476 | 대상 문서 버전 | `target_version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 검토·승인 대상 문서 Revision의 표시 버전 | `v1.0` |
| 477 | 상신자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 문서를 검토·승인 절차에 상신한 사용자 | `UUID` |
| 478 | 상신 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 문서가 검토·승인 절차에 상신된 시각 | `2026-08-31T15:00:00` |
| 479 | Workflow 상태 | `workflow_status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | N | N | Y | 전체 Workflow 상태. DRAFT, IN_PROGRESS, APPROVED, REJECTED, CANCELLED | `IN_PROGRESS` |
| 480 | 현재 단계 순서 | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 현재 처리 중인 검토·승인 단계 순서 | `1` |
| 481 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 승인·반려 또는 취소로 Workflow가 종료된 시각 | `2026-08-31T17:00:00` |
| 482 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00` |
| 483 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Workflow 레코드 생성자 | `UUID` |
| 484 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00` |
| 485 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Workflow 레코드 최종 수정자 | `UUID` |
| 486 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-workflow_step"></a>
### 33. Workflow 단계 (`workflow_step`)

| 항목 | 정의 |
|---|---|
| 설명 | Workflow 내 단계별 처리 상태 및 담당자 관리 |
| Primary Key | `workflow_step_id` |
| 주요 참조(FK) | `workflow_instance_id, assignee_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 487 | Workflow 단계 ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Workflow 검토·승인 단계 고유 식별자. 활성 데이터는 (workflow_instance_id, step_order) 조합의 중복 등록을 허용하지 않음 | `UUID` |
| 488 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | 단계가 소속된 Workflow 식별자 | `UUID` |
| 489 | 단계 순서 | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | 동일 Workflow 내 검토·승인 처리 순서 | `1` |
| 490 | 단계 유형 | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 처리 단계 유형. REVIEW 또는 APPROVE | `REVIEW` |
| 491 | 단계명 | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 화면에 표시할 검토·승인 단계명 | `품질 검토` |
| 492 | 담당자 ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 해당 단계를 처리하도록 지정된 사용자 | `UUID` |
| 493 | 단계 상태 | `step_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 단계 상태. PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 494 | 처리 기한 | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계의 검토·승인 처리 예정 기한 | `2026-09-02T18:00:00` |
| 495 | 처리 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계가 승인·반려 등으로 완료된 시각 | `2026-09-01T10:00:00` |
| 496 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00` |
| 497 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Workflow 단계 생성자 | `UUID` |
| 498 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00` |
| 499 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | Workflow 단계 최종 수정자 | `UUID` |
| 500 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-approval_action"></a>
### 34. 승인 처리 이력 (`approval_action`)

| 항목 | 정의 |
|---|---|
| 설명 | 검토·승인·반려 등 단계별 실제 처리 이력 관리 |
| Primary Key | `approval_action_id` |
| 주요 참조(FK) | `workflow_step_id, actor_id, signature_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 501 | 승인 처리 이력 ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 검토·승인 처리 이력 고유 식별자 | `UUID` |
| 502 | Workflow 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | 처리 이력이 발생한 Workflow 단계 | `UUID` |
| 503 | 처리 유형 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 수행한 처리 유형. SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 504 | 처리자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 실제 검토·승인·반려 처리를 수행한 사용자 | `UUID` |
| 505 | 처리 의견 | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | 검토·승인 처리 시 입력한 의견 | `검토 결과 이상 없음` |
| 506 | 반려 사유 | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | REJECT 처리 시 입력하는 반려 사유 | `증적 파일 보완 필요` |
| 507 | 전자서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 승인·반려 처리와 연결된 전자서명 기록 | `UUID` |
| 508 | 처리 시각 | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 상신·검토·승인·반려가 실제 처리된 시각 | `2026-09-01T10:00:00` |
| 509 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-09-01T10:00:00` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Traceability

<a id="table-traceability_link"></a>
### 35. 공통 추적 관계 (`traceability_link`)

| 항목 | 정의 |
|---|---|
| 설명 | URS, FDS, DQ, FRA, IQ, OQ, PQ 항목 간 추적 관계 관리 |
| Primary Key | `traceability_link_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 510 | 추적 관계 ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 산출물 항목 간 추적 관계 고유 식별자. 활성 데이터는 (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) 조합의 중복 등록을 허용하지 않음 | `UUID` |
| 511 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 추적 관계가 속한 프로젝트 ID | `UUID` |
| 512 | 출발 엔터티 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 출발 대상 유형. REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `REQUIREMENT` |
| 513 | 출발 엔터티 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | source_entity_type에 해당하는 테이블의 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `UUID` |
| 514 | 연결 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 연결 대상 유형. REQUIREMENT, FDS_ITEM, DDS_ITEM, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM | `IQ_ITEM` |
| 515 | 연결 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | target_entity_type에 해당하는 테이블의 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `UUID` |
| 516 | 관계 유형 | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 517 | 연결 근거 | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | 두 산출물 항목을 연결한 업무적 근거 | `URS 요구사항을 IQ 시험으로 검증` |
| 518 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 519 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 생성한 사용자 ID | `UUID` |
| 520 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 521 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 최종 수정한 사용자 ID | `UUID` |
| 522 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 추적 관계 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## DDS

<a id="table-dds_spec"></a>
### 41. 상세 설계 명세서 (`dds_spec`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 DDS 문서 헤더, 버전 및 승인 상태 관리 |
| Primary Key | `dds_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 587 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DDS 문서 고유 식별자 | `UUID` |
| 588 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | DDS가 속한 Validation 프로젝트 | `UUID` |
| 589 | DDS 번호 | `dds_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS 문서 번호 | `DDS-VP-SYS-001` |
| 590 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 상세 설계 명세서 제목 | `Detailed Design Specification` |
| 591 | 문서 버전 | `version` | `varchar(20)` | N | N | - | Y | `v1.0` | N | N | N | Y | DDS 문서 표시 버전 | `v1.0` |
| 592 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | DDS 개정 순번 | `1` |
| 593 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | DDS 신규 작성 또는 개정 사유 | `최초 작성` |
| 594 | 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 현재 유효한 최신 DDS 버전 여부 | `True` |
| 595 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS 문서 상태. DRAFT, REVIEW, APPROVED, REJECTED | `APPROVED` |
| 596 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 생성 시각(UTC) | `2026-09-01T10:00:00` |
| 597 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 작성 사용자 | `UUID` |
| 598 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 최종 수정 시각(UTC) | `2026-09-01T10:00:00` |
| 599 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 최종 수정 사용자 | `UUID` |
| 600 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-dds_item"></a>
### 42. DDS 상세 항목 (`dds_item`)

| 항목 | 정의 |
|---|---|
| 설명 | DDS의 데이터베이스, 인터페이스 및 컴포넌트 상세 설계 항목 관리 |
| Primary Key | `dds_item_id` |
| 주요 참조(FK) | `dds_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 601 | DDS 항목 ID | `dds_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DDS 상세 항목 고유 식별자 | `UUID` |
| 602 | DDS ID | `dds_id` | `uuid` | N | Y | `dds_spec.dds_id` | Y | - | N | Y | N | Y | 상위 DDS 문서 식별자 | `UUID` |
| 603 | DDS 항목 번호 | `item_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DDS 상세 항목 관리 번호 | `DDS-001` |
| 604 | 설계 유형 | `design_type` | `varchar(30)` | N | N | - | Y | `COMPONENT` | N | Y | N | Y | DATABASE, COMPONENT, INTERFACE, SECURITY, BATCH | `DATABASE` |
| 605 | 설계명 | `design_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 상세 설계 항목명 | `사용자 인증 테이블 설계` |
| 606 | 설계 설명 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | DDS 상세 설계 내용 | `사용자 인증 및 권한 테이블 구조` |
| 607 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `DRAFT` | N | Y | N | Y | DDS 항목 상태. DRAFT, REVIEW, APPROVED | `APPROVED` |
| 608 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 항목 생성 시각(UTC) | `09/01/2026 10:00:00` |
| 609 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 항목 작성 사용자 | `UUID` |
| 610 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 항목 최종 수정 시각(UTC) | `09/01/2026 10:00:00` |
| 611 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 항목 최종 수정 사용자 | `UUID` |
| 612 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | DDS 항목 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Deviation

<a id="table-deviation"></a>
### 43. 일탈 관리 (`deviation`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서 및 시험 수행 중 발생한 일탈과 조사·해결·승인 상태 관리 |
| Primary Key | `deviation_id` |
| 주요 참조(FK) | `project_id, resolved_by, approved_by, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 613 | 일탈 ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 일탈 고유 식별자 | `UUID` |
| 614 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 일탈이 발생한 Validation 프로젝트 | `UUID` |
| 615 | 발생 대상 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 일탈 발생 대상 유형. IQ_ITEM, OQ_ITEM, PQ_ITEM 등 | `OQ_ITEM` |
| 616 | 발생 대상 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 엔터티 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `UUID` |
| 617 | 일탈 번호 | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트 내 일탈 관리 번호 | `DEV-001` |
| 618 | 일탈 제목 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 일탈 제목 | `예상 결과 불일치` |
| 619 | 일탈 설명 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 일탈 내용 및 발생 상황 | `OQ 수행 중 예상 결과와 실제 결과 불일치` |
| 620 | 심각도 | `severity` | `varchar(20)` | N | N | - | Y | `MINOR` | N | Y | N | Y | 일탈 심각도. MINOR, MAJOR, CRITICAL | `MAJOR` |
| 621 | 일탈 상태 | `deviation_status` | `varchar(20)` | N | N | - | Y | `OPEN` | N | Y | N | Y | OPEN, INVESTIGATING, RESOLVED, CLOSED, CANCELLED | `OPEN` |
| 622 | 해결 내용 | `resolution` | `text` | N | N | - | N | - | N | N | N | Y | 일탈 원인 조사 및 해결 내용 | `설정값 수정 후 재시험 완료` |
| 623 | 해결 시각 | `resolved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 일탈 해결 완료 시각 | `09/03/2026 15:00:00` |
| 624 | 해결자 ID | `resolved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 일탈 해결 처리 사용자 | `UUID` |
| 625 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 일탈 종결 승인 시각 | `09/03/2026 17:00:00` |
| 626 | 승인자 ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 일탈 종결 승인 사용자 | `UUID` |
| 627 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 생성 시각(UTC) | `09/01/2026 10:00:00` |
| 628 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 등록 사용자 | `UUID` |
| 629 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 최종 수정 시각(UTC) | `09/01/2026 10:00:00` |
| 630 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 최종 수정 사용자 | `UUID` |
| 631 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 일탈 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Report

<a id="table-report_generation"></a>
### 44. 리포트 생성 작업 (`report_generation`)

| 항목 | 정의 |
|---|---|
| 설명 | Audit 및 운영 리포트의 생성 요청, 실행 상태, 실패, 재시도 및 결과 파일 관리 |
| Primary Key | `report_generation_id` |
| 주요 참조(FK) | `project_id, requested_by, result_file_id, report_schedule_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 리포트 생성 결과 및 실행이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 632 | 리포트 생성 ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 리포트 생성 작업 고유 식별자 | `UUID` |
| 633 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 단위 리포트인 경우 연결하며 전체 시스템 또는 조직 단위 리포트는 NULL 허용 | `UUID` |
| 634 | 리포트 유형 | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 리포트 유형. AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 635 | 리포트명 | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 사용자에게 표시되는 생성 리포트명 | `2026년 8월 Audit Trail 리포트` |
| 636 | 실행 방식 | `execution_type` | `varchar(20)` | N | N | - | Y | `ON_DEMAND` | N | Y | N | Y | 실행 방식. ON_DEMAND는 사용자 요청, SCHEDULED는 정기 배치 실행 | `ON_DEMAND` |
| 637 | 조회 시작일시 | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 시작일시. 조회기간이 없는 리포트는 NULL 허용 | `2026-08-01 00:00:00+00` |
| 638 | 조회 종료일시 | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 종료일시. period_from보다 빠를 수 없음 | `2026-08-31 23:59:59+00` |
| 639 | 조회 조건 | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | 조직, 프로젝트, 사용자, 작업 유형, 상태 등 리포트 생성 조건을 JSON으로 저장 | `{"action_types":["CREATE","UPDATE"]}` |
| 640 | 출력 형식 | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | 출력 파일 형식. PDF, XLSX, CSV | `PDF` |
| 641 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 처리 상태. PENDING, PROCESSING, COMPLETED, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 642 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 요청 시 요청자 ID. 시스템 또는 정기 배치 생성 시 NULL 허용 | `UUID` |
| 643 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 리포트 생성 요청이 접수되거나 배치 작업이 등록된 시각 | `2026-09-02 15:00:00+00` |
| 644 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 생성 Worker가 실제 작업을 시작한 시각 | `2026-09-02 15:00:05+00` |
| 645 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 생성 성공 또는 최종 실패로 작업이 종료된 시각 | `2026-09-02 15:01:30+00` |
| 646 | 결과 파일 ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | 생성 완료된 리포트 파일 ID. COMPLETED 상태에서는 필수이며 완료 전에는 NULL 허용 | `UUID` |
| 647 | 재시도 횟수 | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 최초 실행 실패 후 수행한 재시도 횟수. 0 이상이어야 함 | `0` |
| 648 | 최대 재시도 횟수 | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | 자동 재시도 최대 허용 횟수. 0 이상이어야 함 | `3` |
| 649 | 다음 재시도 시각 | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | RETRY_WAIT 상태 작업의 다음 실행 예정 시각 | `2026-09-02 15:10:00+00` |
| 650 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인을 분류하는 시스템 오류 코드 | `REPORT_FILE_CREATE_FAILED` |
| 651 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 리포트 생성 실패 상세 내용. 비밀번호, 토큰 등 민감정보는 저장하지 않음 | `결과 파일 저장 중 오류 발생` |
| 652 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 레코드 생성 시각(UTC) | `2026-09-02 15:00:00+00` |
| 653 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자가 생성한 경우 사용자 ID를 저장하며 시스템·배치가 생성한 경우 NULL 허용 | `UUID` |
| 654 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 최종 수정 시각(UTC) | `2026-09-02 15:01:30+00` |
| 655 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 수정 시 사용자 ID를 저장하며 시스템·배치 처리 시 NULL 허용 | `UUID` |
| 656 | 리포트 일정 ID | `report_schedule_id` | `uuid` | N | Y | `report_schedule.report_schedule_id` | N | - | N | Y | N | Y | 정기 실행으로 생성된 경우 원본 리포트 일정 ID 저장. 사용자 요청 실행은 NULL 허용 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-report_schedule"></a>
### 50. 리포트 실행 일정 (`report_schedule`)

| 항목 | 정의 |
|---|---|
| 설명 | Audit 및 운영 리포트의 실행주기, 조회기간, 출력형식, 다음 실행시각 및 활성 상태 관리 |
| Primary Key | `report_schedule_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 실행 일정과 변경이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 744 | 리포트 일정 ID | `report_schedule_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 정기 리포트 실행 일정 고유 식별자 | `UUID` |
| 745 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 단위 리포트 일정인 경우 연결하며 조직·시스템 단위는 NULL 허용 | `UUID` |
| 746 | 일정명 | `schedule_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 사용자에게 표시되는 정기 리포트 일정명 | `월간 Audit Trail 리포트` |
| 747 | 리포트 유형 | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 리포트 유형. AUDIT_TRAIL, PROJECT_STATUS, WORKFLOW_STATUS, DEVIATION_STATUS, TRACEABILITY, SYSTEM_OPERATION | `AUDIT_TRAIL` |
| 748 | 실행주기 유형 | `schedule_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 실행주기 유형. DAILY, WEEKLY, MONTHLY, CRON | `MONTHLY` |
| 749 | 실행주기 설정 | `schedule_expression` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 실행일·요일·시각 또는 Cron 표현식 등 실행주기 설정값 | `0 0 1 * *` |
| 750 | 조회기간 유형 | `period_type` | `varchar(30)` | N | N | - | Y | `PREVIOUS_MONTH` | N | N | N | Y | 원천 데이터 조회기간 산정 기준. PREVIOUS_DAY, PREVIOUS_WEEK, PREVIOUS_MONTH, CUSTOM | `PREVIOUS_MONTH` |
| 751 | 조회 조건 | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | N | Y | 조직, 프로젝트, 사용자, 작업유형, 상태 등 정기 리포트 조회 조건 | `{"action_types":["CREATE","UPDATE"]}` |
| 752 | 출력 형식 | `output_format` | `varchar(20)` | N | N | - | Y | `PDF` | N | Y | N | Y | 출력 파일 형식. PDF, XLSX, CSV | `PDF` |
| 753 | 다음 실행 시각 | `next_run_at` | `timestamptz` | N | N | - | Y | - | N | Y | N | Y | 해당 일정이 다음으로 실행될 예정 시각 | `2026-10-01 00:00:00+00` |
| 754 | 마지막 실행 시각 | `last_run_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 일정이 마지막으로 실행된 시각 | `2026-09-01 00:00:00+00` |
| 755 | 마지막 생성 작업 ID | `last_report_generation_id` | `uuid` | N | Y | `report_generation.report_generation_id` | N | - | N | Y | N | Y | 해당 일정으로 가장 최근 생성된 리포트 작업 | `UUID` |
| 756 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `True` | N | Y | N | Y | 정기 리포트 일정의 활성 여부 | `True` |
| 757 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 일정 생성 시각(UTC) | `2026-09-02 18:00:00+00` |
| 758 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 리포트 일정을 등록한 사용자 | `UUID` |
| 759 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 일정 최종 수정 시각(UTC) | `2026-09-02 18:00:00+00` |
| 760 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 리포트 일정을 최종 수정한 사용자 | `UUID` |
| 761 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 일정 소프트 삭제 시각 | - |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## AI

<a id="table-ai_generation_job"></a>
### 45. AI 생성 작업 (`ai_generation_job`)

| 항목 | 정의 |
|---|---|
| 설명 | AI 생성 요청, 모델, 입력조건, 실행상태, 실패·재시도 관리 |
| Primary Key | `ai_job_id` |
| 주요 참조(FK) | `project_id, requested_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 657 | AI 작업 ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 생성 작업 식별자 | `UUID` |
| 658 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 연결 | `UUID` |
| 659 | 작업 유형 | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 660 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM, OQ_REPORT, PQ_REPORT, VSR_REPORT | `REQUIREMENT` |
| 661 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 생성 대상 식별자 | `UUID` |
| 662 | AI 모델명 | `model_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | GPT, Claude, Gemini | `GPT-5` |
| 663 | 입력 파라미터 | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 생성 조건 JSON 저장 | `JSON` |
| 664 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | PENDING, PROCESSING, COMPLETED, FAILED, RETRY_WAIT, CANCELLED | `COMPLETED` |
| 665 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 요청 사용자 | `UUID` |
| 666 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 요청 접수 시각 | `2026-09-02T00:00:00` |
| 667 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 시작 시각 | `2026-09-02T00:00:00` |
| 668 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 완료 시각 | `2026-09-02T00:00:00` |
| 669 | 재시도 횟수 | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 재실행 횟수 | `0` |
| 670 | 최대 재시도 횟수 | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | 재시도 상한 | `3` |
| 671 | 다음 재시도 시각 | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 재시도 예정 시각 | `2026-09-02T15:10:00` |
| 672 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인 코드 | `LLM_TIMEOUT` |
| 673 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 실패 상세 메시지 | `모델 응답시간 초과` |
| 674 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00` |
| 675 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 676 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00` |
| 677 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-ai_generation_result"></a>
### 46. AI 생성 결과 (`ai_generation_result`)

| 항목 | 정의 |
|---|---|
| 설명 | AI 생성 결과 집합 및 채택 상태 관리 |
| Primary Key | `ai_result_id` |
| 주요 참조(FK) | `ai_job_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 678 | AI 결과 ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 결과 식별자 | `UUID` |
| 679 | AI 작업 ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | 상위 AI 작업 | `UUID` |
| 680 | 결과 제목 | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | 결과 제목 | `OQ 테스트 초안` |
| 681 | 선택 여부 | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | 사용자 채택 여부 | `True` |
| 682 | 반영 여부 | `is_applied` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | 실제 산출물 반영 여부 | `True` |
| 683 | 선택 사용자 ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 채택 사용자 | `UUID` |
| 684 | 선택 시각 | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 채택 시각 | `2026-09-02T00:00:00` |
| 685 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00` |
| 686 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 687 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00` |
| 688 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

<a id="table-ai_result_item"></a>
### 47. AI 생성 결과 항목 (`ai_result_item`)

| 항목 | 정의 |
|---|---|
| 설명 | AI가 생성한 URS/FRA/IQ/OQ/PQ 항목 및 문서 섹션 상세 관리 |
| Primary Key | `ai_result_item_id` |
| 주요 참조(FK) | `ai_result_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 689 | AI 결과 항목 ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 결과 항목 식별자 | `UUID` |
| 690 | AI 결과 ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | 상위 결과 참조 | `UUID` |
| 691 | 항목 순번 | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 결과 표시 순서 | `1` |
| 692 | 항목 유형 | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 693 | 제목 | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 생성 항목 제목 | `전자서명 기록` |
| 694 | 본문 내용 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | 생성 결과 본문 | `내용` |
| 695 | 적용 대상 유형 | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | REQUIREMENT, FRA_ITEM, IQ_ITEM 등 | `REQUIREMENT` |
| 696 | 적용 대상 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 실제 저장 대상 PK | `UUID` |
| 697 | 채택 여부 | `is_selected` | `boolean` | N | N | - | Y | `N` | N | Y | N | Y | 사용자 채택 여부 | `True` |
| 698 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00` |
| 699 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 700 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00` |
| 701 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---

## Notification

<a id="table-notification_delivery"></a>
### 48. 알림 발송 (`notification_delivery`)

| 항목 | 정의 |
|---|---|
| 설명 | 검토·승인 요청, 처리 지연 및 시스템 업무 알림의 발송 대상, 발송 상태, 실패 및 재시도 이력 관리 |
| Primary Key | `notification_delivery_id` |
| 주요 참조(FK) | `project_id, workflow_instance_id, workflow_step_id, recipient_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 알림 발송 내용과 실행이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 702 | 알림 발송 ID | `notification_delivery_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 알림 발송 작업 고유 식별자 | `UUID` |
| 703 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 관련 알림인 경우 연결하며 시스템 공통 알림은 NULL 허용 | `UUID` |
| 704 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 검토·승인 Workflow 관련 알림인 경우 연결 | `UUID` |
| 705 | Workflow 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | N | - | N | Y | N | Y | 검토·승인 단계 관련 알림인 경우 연결 | `UUID` |
| 706 | 알림 유형 | `notification_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 알림 업무 유형. APPROVAL_REQUEST, REVIEW_REQUEST, DUE_REMINDER, OVERDUE, REJECTION, COMPLETION, SYSTEM | `OVERDUE` |
| 707 | 발송 채널 | `delivery_channel` | `varchar(20)` | N | N | - | Y | `EMAIL` | N | Y | N | Y | 발송 채널. EMAIL, MESSENGER, PUSH, IN_APP | `EMAIL` |
| 708 | 수신자 ID | `recipient_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | Y | Y | 알림을 수신하는 사용자 ID | `UUID` |
| 709 | 알림 제목 | `notification_title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 사용자에게 발송되는 알림 제목 | `검토 처리기한 초과 안내` |
| 710 | 알림 내용 | `notification_content` | `text` | N | N | - | Y | - | N | N | N | Y | 사용자에게 발송되는 알림 본문. 비밀번호, 토큰 등 민감정보 저장 금지 | `FDS 검토 처리기한이 초과되었습니다.` |
| 711 | 발송 상태 | `delivery_status` | `varchar(20)` | N | N | - | Y | `PENDING` | N | Y | N | Y | 발송 상태. PENDING, PROCESSING, SENT, RETRY_WAIT, FAILED, CANCELLED | `PENDING` |
| 712 | 발송 예정 시각 | `scheduled_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 알림 발송 예정 시각. 즉시 발송은 NULL 허용 | `2026-09-02 18:00:00+00` |
| 713 | 발송 시각 | `sent_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 외부 발송 채널에 정상 전달된 시각 | `2026-09-02 18:00:05+00` |
| 714 | 재시도 횟수 | `retry_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 최초 발송 실패 이후 재시도한 횟수. 0 이상 | `0` |
| 715 | 최대 재시도 횟수 | `max_retry_count` | `integer` | N | N | - | Y | `3` | N | N | N | Y | 자동 발송 재시도 최대 허용 횟수. 0 이상 | `3` |
| 716 | 다음 재시도 시각 | `next_retry_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | RETRY_WAIT 상태 알림의 다음 발송 예정 시각 | `2026-09-02 18:10:00+00` |
| 717 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 알림 발송 실패 원인을 분류하는 시스템 오류 코드 | `EMAIL_SEND_TIMEOUT` |
| 718 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 알림 발송 실패 상세 내용. 인증정보 등 민감정보 저장 금지 | `메일 서버 응답시간 초과` |
| 719 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 알림 발송 작업 생성 시각(UTC) | `2026-09-02 18:00:00+00` |
| 720 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 생성 시 사용자 ID를 저장하며 시스템·배치 생성 시 NULL 허용 | `UUID` |
| 721 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 알림 발송 작업 최종 수정 시각(UTC) | `2026-09-02 18:00:05+00` |
| 722 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 수정 시 사용자 ID를 저장하며 시스템·배치 처리 시 NULL 허용 | `UUID` |

[↑ 맨 위로](#dvt-데이터-테이블-정의서)

---
