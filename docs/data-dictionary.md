<a id="top"></a>

# 데이터 테이블 정의서

> Validation Management Platform 데이터 모델 문서  
> **예시값 기준**: 개인정보 및 민감정보는 비식별 샘플 값으로 표기

## 목차

- [1. 문서 개요](#1-문서-개요)
- [2. 표기 기준](#2-표기-기준)
- [3. 테이블 목록](#3-테이블-목록)
- [4. 도메인별 바로가기](#4-도메인별-바로가기)
- [5. 상세 테이블 및 컬럼 정의](#5-상세-테이블-및-컬럼-정의)

## 1. 문서 개요

- 전체 테이블: **73개**
- 전체 컬럼: **1,177개**
- 도메인: **26개**
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
| UQ | 개별 컬럼의 유일성 여부. 복합·조건부 유일성은 해당 테이블의 제약조건에 별도 정의 |
| IDX | 인덱스 지정 여부. PK·개별 UNIQUE가 생성하는 인덱스도 Y로 표기하며 복합·조건부 인덱스는 제약조건을 함께 확인 |
| 민감 | 개인/민감정보 구분 |
| Default | SQL 표현식. 문자열은 작은따옴표, Boolean은 TRUE/FALSE, 함수는 함수식 그대로 표기하며 `-`는 기본값 미지정 |

`date` 예시는 `YYYY-MM-DD`, `timestamptz` 예시는 UTC 기준 `YYYY-MM-DDTHH:mm:ssZ`로 표기합니다.

복합·조건부 유일성의 구성 컬럼을 각각 `UQ=Y`로 표기하지 않습니다. 활성 데이터의 판정은 해당 제약조건의 `WHERE` 조건을 따릅니다.

민감=Y인 값은 허용된 사용자·처리 목적에 한해 조회·내보내기한다. 실명, 이메일, IP, 감사자·승인자 성명 및 개인정보를 포함할 수 있는 Audit JSON을 이 분류에 포함한다. password_hash는 응답·일반 내보내기·Audit JSON에서 제외하고 Audit=N을 유지한다.

NN=N은 승인·완료 시에도 항상 선택이라는 뜻은 아닙니다. 조건부 필수값은 해당 테이블의 제약조건·업무 규칙을 따릅니다. 복수 테이블을 가리키는 유형+ID 및 JSON 내부 참조는 실제 FK와 구분합니다.

DB의 PK·FK·CHECK·UNIQUE는 저장 시 강제하는 제약이다. 제약조건 표는 한 행에 하나의 DB 제약 또는 업무 검증을 정의한다. 조건부 UNIQUE는 해당 WHERE 조건의 행에만 적용한다. 업무 검증은 서버에서 수행하고 UI의 선택 제한만으로 대체하지 않는다.

참조 저장·상신·승인 시 대상 존재, 부모·프로젝트·프로토콜 개정의 일치 및 현재 사용·승인 가능 상태를 검사한다. 다형 참조와 JSON 내부 참조는 허용된 실제 테이블·PK·버전과 객체 형식을 검사한다. 프로젝트 업무끼리의 연결은 소속 프로젝트 일치를 요구하고 공용 규정·라이브러리 등 프로젝트 외 자료는 해당 자료의 조회 및 사용 권한을 검사한다. 신규 업무 연결에는 삭제된 대상과 다른 프로젝트의 업무 기록을 허용하지 않는다. 과거 승인 참조의 조회는 당시 원문을 유지하고 이후의 폐기·유효성 상실 상태와 구분한다.

동시 요청은 서버 트랜잭션과 대상 행 잠금 또는 저장 버전 비교로 제어한다. 중복 승인·번호 발급·최신 개정 전환·진행 중 종료 요청 생성은 검사와 저장을 분리하지 않는다. 저장 버전이 바뀐 요청은 충돌로 처리하고 이전 화면의 값으로 최신 데이터를 덮어쓰지 않는다. 상태 전환·서명·감사기록은 함께 확정하거나 함께 취소하며 감사 변경 필드를 임의로 생략하지 않는다.

## 3. 테이블 목록

| No | Domain | 논리 테이블명 | 물리 테이블명 | PK | 주요 참조(FK) | GxP | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | 조직/고객사 | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | 사용자 | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | 역할 | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | 사용자 역할 | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id` | High | Y |
| 5 | Security | 사용자 그룹 | [`user_group`](#table-user_group) | `user_group_id` | `organization_id, created_by, updated_by` | High | Y |
| 6 | Security | 사용자 그룹 구성원 | [`user_group_member`](#table-user_group_member) | `user_group_member_id` | `user_group_id, user_id, created_by, updated_by` | High | Y |
| 7 | Security | 사용자 그룹 역할 | [`group_role`](#table-group_role) | `group_role_id` | `user_group_id, role_id, project_id, created_by, updated_by` | High | Y |
| 8 | Security | 메뉴·프로젝트 접근권한 | [`access_permission_grant`](#table-access_permission_grant) | `grant_id` | `user_id, user_group_id, project_id, created_by, updated_by` | High | Y |
| 9 | Security | 인벤토리 업무 역할 | [`inventory_role_grant`](#table-inventory_role_grant) | `grant_id` | `user_id, user_group_id, created_by, updated_by` | High | Y |
| 10 | Compliance | 전자서명 | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 11 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 12 | File | 파일 자산 | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 13 | File | 파일 악성코드 검사 작업 | [`file_scan_job`](#table-file_scan_job) | `scan_job_id` | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` | High | Y |
| 14 | File | 증적 파일 연결 | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 15 | System | 시스템/장비 식별 정보 | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 16 | System | 시스템 인벤토리 개정 이력 | [`system_asset_revision`](#table-system_asset_revision) | `system_revision_id` | `system_id, processed_by, signature_id` | High | Y |
| 17 | Library | 라이브러리 항목 마스터 | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 18 | Validation | Validation 프로젝트 | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` | High | Y |
| 19 | Validation | 프로젝트 검증 대상 개정 | [`project_system_baseline`](#table-project_system_baseline) | `baseline_id` | `project_id, system_revision_id, adopted_by` | Critical | Y |
| 20 | Validation | 프로젝트 참여자 | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 21 | Validation | 밸리데이션 활동 마스터 | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 22 | Validation | 프로젝트 수행 활동 | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 23 | Validation | 활동 선후행 조건 | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 24 | Validation | 프로젝트 종료 요청 | [`project_closure_request`](#table-project_closure_request) | `closure_request_id` | `project_id, requested_by, workflow_instance_id, created_by, updated_by` | High | Y |
| 25 | VP | 검증 계획 | [`vp_plan`](#table-vp_plan) | `vp_id` | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 26 | VP | 검증 계획 목차 | [`vp_section`](#table-vp_section) | `vp_section_id` | `vp_id, created_by, updated_by` | High | Y |
| 27 | QIA | 품질 영향 평가 헤더 | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id, created_by, updated_by` | High | Y |
| 28 | QIA | QIA 모듈 상세 평가 | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 29 | QIA | QIA 프로세스 평가 | [`qia_process`](#table-qia_process) | `qia_process_id` | `qia_module_item_id, created_by, updated_by` | High | Y |
| 30 | VA | 공급업체 감사 평가 | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 31 | URS | 사용자 요구사항 명세 | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` | High | Y |
| 32 | FDS | 기능 설계 명세서 | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 33 | DDS | 상세 설계 명세서 | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 34 | DQ | 설계 적격성 평가 | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 35 | DQ | DQ 상세 평가 항목 | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` | High | Y |
| 36 | FRA | 기능 위험평가 | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 37 | FRA | FRA 위험 상세 항목 | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` | High | Y |
| 38 | IQ | 설치 적격성 평가 | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 39 | IQ | IQ 상세 테스트 항목 | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 40 | IQ | IQ 시험 절차 | [`iq_step`](#table-iq_step) | `step_id` | `iq_item_id, created_by, updated_by` | High | Y |
| 41 | IQ | IQ 시험 수행 | [`iq_execution`](#table-iq_execution) | `execution_id` | `iq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 42 | IQ | IQ 절차 수행 | [`iq_step_execution`](#table-iq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 43 | OQ | 운전 적격성 평가 | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 44 | OQ | OQ 상세 테스트 항목 | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 45 | OQ | OQ 시험 절차 | [`oq_step`](#table-oq_step) | `step_id` | `oq_item_id, created_by, updated_by` | High | Y |
| 46 | OQ | OQ 시험 수행 | [`oq_execution`](#table-oq_execution) | `execution_id` | `oq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 47 | OQ | OQ 절차 수행 | [`oq_step_execution`](#table-oq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 48 | PQ | 성능 적격성 평가 | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 49 | PQ | PQ 상세 테스트 항목 | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 50 | PQ | PQ 시험 절차 | [`pq_step`](#table-pq_step) | `step_id` | `pq_item_id, created_by, updated_by` | High | Y |
| 51 | PQ | PQ 시험 수행 | [`pq_execution`](#table-pq_execution) | `execution_id` | `pq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 52 | PQ | PQ 절차 수행 | [`pq_step_execution`](#table-pq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 53 | VSR | 밸리데이션 종합 보고서 | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` | Critical | Y |
| 54 | VSR | VSR 활동 요약 항목 | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` | Critical | Y |
| 55 | Workflow | Workflow 인스턴스 | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by, project_id, workflow_config_id` | Critical | Y |
| 56 | Workflow | Workflow 단계 | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, created_by, updated_by` | Critical | Y |
| 57 | Workflow | 결재 단계 담당자 | [`workflow_step_assignee`](#table-workflow_step_assignee) | `assignment_id` | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` | Critical | Y |
| 58 | Workflow | 승인 처리 이력 | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` | Critical | Y |
| 59 | Workflow | 프로젝트 결재선 설정 | [`project_workflow_config`](#table-project_workflow_config) | `config_id` | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` | Critical | Y |
| 60 | Traceability | 공통 추적 관계 | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 61 | Deviation | 일탈 관리 | [`deviation`](#table-deviation) | `deviation_id` | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id` | Critical | Y |
| 62 | Deviation | 일탈 조치 회차 | [`deviation_action_round`](#table-deviation_action_round) | `action_round_id` | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 63 | Document | 산출물 문서 | [`deliverable_document`](#table-deliverable_document) | `document_id` | `project_id, project_activity_id, created_by, updated_by` | Critical | Y |
| 64 | Document | 산출물 개정 | [`deliverable_revision`](#table-deliverable_revision) | `document_revision_id` | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` | Critical | Y |
| 65 | Document | 산출물 목차/본문 | [`deliverable_section`](#table-deliverable_section) | `section_id` | `document_revision_id, created_by, updated_by` | Critical | Y |
| 66 | Report | 리포트 생성 작업 | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` | High | Y |
| 67 | AI | AI 생성 작업 | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 68 | AI | AI 생성 결과 | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, selected_by, created_by, updated_by` | High | Y |
| 69 | AI | AI 생성 결과 항목 | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 70 | Regulation | 규정 근거 문서 | [`regulatory_source`](#table-regulatory_source) | `regulatory_source_id` | `file_id, verified_by, created_by, updated_by` | High | Y |
| 71 | Regulation | 규정 조항 | [`regulatory_clause`](#table-regulatory_clause) | `regulatory_clause_id` | `regulatory_source_id, created_by, updated_by` | High | Y |
| 72 | Regulation | 요구사항 규정 근거 | [`requirement_regulation`](#table-requirement_regulation) | `requirement_regulation_id` | `requirement_id, regulatory_clause_id, created_by, updated_by` | High | Y |
| 73 | Regulation | 라이브러리 규정 근거 | [`library_item_regulation`](#table-library_item_regulation) | `library_item_regulation_id` | `library_id, regulatory_clause_id, created_by, updated_by` | High | Y |

## 4. 도메인별 바로가기

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role), [`user_group`](#table-user_group), [`user_group_member`](#table-user_group_member), [`group_role`](#table-group_role), [`access_permission_grant`](#table-access_permission_grant), [`inventory_role_grant`](#table-inventory_role_grant)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`file_scan_job`](#table-file_scan_job), [`evidence_link`](#table-evidence_link)
- **System**: [`system_asset`](#table-system_asset), [`system_asset_revision`](#table-system_asset_revision)
- **Library**: [`library_item`](#table-library_item)
- **Validation**: [`validation_project`](#table-validation_project), [`project_system_baseline`](#table-project_system_baseline), [`project_member`](#table-project_member), [`validation_activity`](#table-validation_activity), [`project_activity`](#table-project_activity), [`activity_dependency`](#table-activity_dependency), [`project_closure_request`](#table-project_closure_request)
- **VP**: [`vp_plan`](#table-vp_plan), [`vp_section`](#table-vp_section)
- **QIA**: [`qia_assessment`](#table-qia_assessment), [`qia_module_item`](#table-qia_module_item), [`qia_process`](#table-qia_process)
- **VA**: [`vendor_audit`](#table-vendor_audit)
- **URS**: [`requirement`](#table-requirement)
- **FDS**: [`fds_spec`](#table-fds_spec)
- **DDS**: [`dds_spec`](#table-dds_spec)
- **DQ**: [`dq_assessment`](#table-dq_assessment), [`dq_item`](#table-dq_item)
- **FRA**: [`fra_assessment`](#table-fra_assessment), [`fra_item`](#table-fra_item)
- **IQ**: [`iq_assessment`](#table-iq_assessment), [`iq_item`](#table-iq_item), [`iq_step`](#table-iq_step), [`iq_execution`](#table-iq_execution), [`iq_step_execution`](#table-iq_step_execution)
- **OQ**: [`oq_assessment`](#table-oq_assessment), [`oq_item`](#table-oq_item), [`oq_step`](#table-oq_step), [`oq_execution`](#table-oq_execution), [`oq_step_execution`](#table-oq_step_execution)
- **PQ**: [`pq_assessment`](#table-pq_assessment), [`pq_item`](#table-pq_item), [`pq_step`](#table-pq_step), [`pq_execution`](#table-pq_execution), [`pq_step_execution`](#table-pq_step_execution)
- **VSR**: [`vsr_assessment`](#table-vsr_assessment), [`vsr_item`](#table-vsr_item)
- **Workflow**: [`workflow_instance`](#table-workflow_instance), [`workflow_step`](#table-workflow_step), [`workflow_step_assignee`](#table-workflow_step_assignee), [`approval_action`](#table-approval_action), [`project_workflow_config`](#table-project_workflow_config)
- **Traceability**: [`traceability_link`](#table-traceability_link)
- **Deviation**: [`deviation`](#table-deviation), [`deviation_action_round`](#table-deviation_action_round)
- **Document**: [`deliverable_document`](#table-deliverable_document), [`deliverable_revision`](#table-deliverable_revision), [`deliverable_section`](#table-deliverable_section)
- **Report**: [`report_generation`](#table-report_generation)
- **AI**: [`ai_generation_job`](#table-ai_generation_job), [`ai_generation_result`](#table-ai_generation_result), [`ai_result_item`](#table-ai_result_item)
- **Regulation**: [`regulatory_source`](#table-regulatory_source), [`regulatory_clause`](#table-regulatory_clause), [`requirement_regulation`](#table-requirement_regulation), [`library_item_regulation`](#table-library_item_regulation)

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
| 1 | 조직 ID | `organization_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 조직 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 2 | 조직 코드 | `organization_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | 조직/회사 식별 코드 | `ORG-SAMPLE-01` |
| 3 | 조직명 | `organization_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 회사/사업장명 | `샘플 주식회사` |
| 4 | 조직 유형 | `organization_type` | `varchar(50)` | N | N | - | Y | `'본사'` | N | N | N | Y | 본사 \| 공장 \| 연구소 \| 해외법인 | `본사` |
| 5 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 6 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 조직 설명 | `기업 IT 및 데이터 운영 조직` |
| 7 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 8 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00Z` |

#### 업무 규칙

사용자·그룹·시스템의 소속 조직을 관리한다.

[↑ 맨 위로](#top)

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
| 9 | 사용자 ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 계정 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 10 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | organization.organization_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 11 | 비밀번호 해시 | `password_hash` | `varchar(255)` | N | N | - | Y | - | N | N | Y | N | 비밀번호 검증용 해시값. 응답·일반 내보내기·감사로그에 해시 원문을 포함하지 않는다. Audit=N을 유지한다. | `[REDACTED_PASSWORD_HASH]` |
| 12 | 사용자 실명 | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | Y | Y | 사용자 이름 | `홍길동` |
| 13 | 이메일 | `email` | `varchar(254)` | N | N | - | Y | - | Y | Y | Y | Y | 계정 초대·로그인에 사용하는 이메일. 화면의 계정 값이며 별도 로그인 ID를 두지 않는다. | `user@example.com` |
| 14 | 관리 권한등급 | `permission_level` | `varchar(20)` | N | N | - | Y | `'USER'` | N | N | N | Y | GLOBAL_ADMIN=전체 관리자, PROJECT_ADMIN=프로젝트 관리자, USER=사용자. 작성자·검토자 등의 역할과 구분 | `USER` |
| 15 | 부서명 | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 소속 부서명 | `정보전략팀` |
| 16 | 직급/직책 | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 직급 정보 | `선임` |
| 17 | 계정 상태 | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 18 | 최종 로그인 시각 | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 시스템 접속 타임스탬프 | `2026-08-26T16:00:00Z` |
| 19 | 비밀번호 변경일 | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 비밀번호 마지막 변경 시각 | `2026-08-01T09:00:00Z` |
| 20 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 21 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_app_user_1` | 업무 검증 | - | 이메일은 앞뒤 공백 제거 및 대소문자 정규화 후 중복을 허용하지 않는다. |
| `ck_app_user_2` | CHECK | `permission_level` | CHECK (permission_level IN ('GLOBAL_ADMIN','PROJECT_ADMIN','USER')) |

#### 업무 규칙

복수 업무 역할은 user_role, 복수 소속 그룹은 user_group_member에서 관리한다. 화면의 활성/비활성은 ACTIVE/INACTIVE에 대응한다. 계정 잠금 상태, 비밀번호 정보 및 접속 시각을 관리한다.

로그인 식별자는 이메일을 사용한다. 관리 권한등급과 메뉴·프로젝트별 접근권한은 별도로 관리한다.

[↑ 맨 위로](#top)

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
| 23 | 역할 코드 | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | SYSTEM_ADMIN=시스템 관리자, ADMIN=어드민, AUTHOR=작성자, REVIEWER=검토자, APPROVER=승인자, VIEWER=뷰어 | `AUTHOR` |
| 24 | 역할명 | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 계정 화면의 업무 역할 표시명. 사용자 1명에게 여러 역할을 부여할 수 있다. | `전체관리자` |
| 25 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 역할 상세 권한 범위 설명 | `시스템 전체 관리 권한` |
| 26 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |

#### 업무 규칙

메뉴/프로젝트 조회·편집·폐기는 access_permission_grant, 인벤토리 업무 역할은 inventory_role_grant에 저장한다.

[↑ 맨 위로](#top)

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
| 27 | 매핑 ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자-역할 매핑 고유 식별자. 이 테이블의 모든 행에 (user_id, role_id) 복합 UNIQUE를 적용한다. 별도 활성·삭제 컬럼은 없다. | `00000000-0000-0000-0000-000000000001` |
| 28 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 29 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | role.role_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 30 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_role_1` | UNIQUE | `user_id, role_id` | UNIQUE (user_id, role_id) |

[↑ 맨 위로](#top)

---

<a id="table-user_group"></a>
### 5. 사용자 그룹 (`user_group`)

| 항목 | 정의 |
|---|---|
| 설명 | 조직별 사용자 그룹의 기본정보 및 활성 상태 관리 |
| Primary Key | `user_group_id` |
| 주요 참조(FK) | `organization_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 31 | 사용자 그룹 ID | `user_group_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹 고유 식별자 | `UUID` |
| 32 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | 사용자 그룹이 소속된 조직 | `UUID` |
| 33 | 그룹 코드 | `group_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 조직 내 사용자 그룹 식별 코드. deleted_at IS NULL이고 is_active가 TRUE인 행에 (organization_id, group_code) 중복을 허용하지 않는다. | `QA_REVIEWER_GROUP` |
| 34 | 그룹명 | `group_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 사용자에게 표시되는 그룹명 | `QA 검토자 그룹` |
| 35 | 그룹 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 그룹의 목적과 권한 범위 설명 | `Validation 문서 QA 검토 담당자 그룹` |
| 36 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 사용자 그룹 활성 여부 | `TRUE` |
| 37 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자 그룹 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 38 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 사용자 그룹을 생성한 사용자 | `UUID` |
| 39 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자 그룹 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 40 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 사용자 그룹을 최종 수정한 사용자 | `UUID` |
| 41 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 사용자 그룹 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_group_1` | UNIQUE | `organization_id, group_code, deleted_at, is_active` | UNIQUE (organization_id, group_code) WHERE deleted_at IS NULL AND is_active=TRUE |

[↑ 맨 위로](#top)

---

<a id="table-user_group_member"></a>
### 6. 사용자 그룹 구성원 (`user_group_member`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자와 사용자 그룹 간 N:M 관계 및 그룹 참여 상태·기간 관리 |
| Primary Key | `user_group_member_id` |
| 주요 참조(FK) | `user_group_id, user_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 42 | 그룹 구성원 ID | `user_group_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹 구성원 매핑 고유 식별자 | `UUID` |
| 43 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | 구성원이 소속된 사용자 그룹 | `UUID` |
| 44 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹에 포함되는 사용자. deleted_at IS NULL이고 member_status가 ACTIVE인 행에 (user_group_id, user_id) 중복을 허용하지 않는다. | `UUID` |
| 45 | 구성원 상태 | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | 그룹 구성원 상태. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 46 | 참여 시작 시각 | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자가 그룹에 포함된 시각 | `2026-09-14T09:00:00Z` |
| 47 | 참여 종료 시각 | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 사용자의 그룹 참여가 종료된 시각. 현재 구성원이면 NULL | `2026-09-01T00:00:00Z` |
| 48 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 구성원 매핑 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 49 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 구성원을 등록한 사용자 | `UUID` |
| 50 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 구성원 매핑 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 51 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 구성원 매핑을 최종 수정한 사용자 | `UUID` |
| 52 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 그룹 구성원 매핑 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_group_member_1` | UNIQUE | `user_group_id, user_id, deleted_at, member_status` | UNIQUE (user_group_id, user_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |

#### 업무 규칙

한 사용자의 여러 그룹 가입을 허용한다. 그룹별 권한은 활성 소속 그룹에서 합산 조회한다.

[↑ 맨 위로](#top)

---

<a id="table-group_role"></a>
### 7. 사용자 그룹 역할 (`group_role`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자 그룹에 역할을 부여하고 전역 또는 프로젝트별 적용 범위 관리 |
| Primary Key | `group_role_id` |
| 주요 참조(FK) | `user_group_id, role_id, project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 53 | 그룹 역할 ID | `group_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹과 역할 간 권한 매핑 고유 식별자 | `UUID` |
| 54 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | 역할을 부여받는 사용자 그룹 | `UUID` |
| 55 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | 그룹에 부여하는 역할 | `UUID` |
| 56 | 권한 범위 유형 | `scope_type` | `varchar(20)` | N | N | - | Y | `'GLOBAL'` | N | Y | N | Y | 그룹 역할 적용 범위. GLOBAL, PROJECT | `PROJECT` |
| 57 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | scope_type이 PROJECT인 경우 필수. GLOBAL인 경우 NULL. 범위와 NULL 대응은 CHECK 제약으로 검증한다 | `UUID` |
| 58 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 그룹 역할 매핑 사용 여부. deleted_at IS NULL이고 is_active가 TRUE인 행에 범위별 중복 금지를 적용한다. GLOBAL은 (user_group_id, role_id), PROJECT는 (user_group_id, role_id, project_id)를 기준으로 한다. | `TRUE` |
| 59 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 역할 매핑 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 60 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 역할을 부여한 사용자 | `UUID` |
| 61 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 역할 매핑 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 62 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 역할 매핑을 최종 수정한 사용자 | `UUID` |
| 63 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 그룹 역할 매핑 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_group_role_1` | CHECK | `scope_type, project_id` | CHECK ((scope_type='GLOBAL' AND project_id IS NULL) OR (scope_type='PROJECT' AND project_id IS NOT NULL)) |

#### 업무 규칙

그룹의 업무 역할 매핑이다. 화면의 메뉴별·프로젝트별 권한 체크박스는 access_permission_grant에서 관리한다.

[↑ 맨 위로](#top)

---

<a id="table-access_permission_grant"></a>
### 8. 메뉴·프로젝트 접근권한 (`access_permission_grant`)

| 항목 | 정의 |
|---|---|
| 설명 | 개인 또는 그룹에 부여하는 화면의 조회·편집·폐기 권한 |
| Primary Key | `grant_id` |
| 주요 참조(FK) | `user_id, user_group_id, project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 64 | 권한 ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 65 | 개인 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 66 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | user_group.user_group_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 67 | 권한 범위 | `scope_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MENU=메뉴, PROJECT=프로젝트 | `MENU` |
| 68 | 메뉴 코드 | `menu_code` | `varchar(40)` | N | N | - | N | - | N | N | N | Y | SYSTEM_INVENTORY/PROJECT_MANAGEMENT/LIBRARY/AUDIT_TRAIL/ACCOUNT_PERMISSION. MENU 범위에서 필수 | `LIBRARY` |
| 69 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | PROJECT 범위에서 필수. MENU 범위에서는 NULL | `00000000-0000-0000-0000-000000000001` |
| 70 | 조회 허용 | `can_view` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 조회 체크박스 | `FALSE` |
| 71 | 편집 허용 | `can_edit` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 편집 체크박스 | `FALSE` |
| 72 | 폐기 허용 | `can_dispose` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 프로젝트 권한 화면의 폐기 체크박스. 메뉴 범위에서는 FALSE | `FALSE` |
| 73 | 활성 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 권한 회수 시 FALSE로 변경 | `TRUE` |
| 74 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 75 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 76 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 77 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_access_permission_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_access_permission_grant_2` | CHECK | `scope_type, menu_code, project_id, can_dispose` | CHECK ((scope_type='MENU' AND menu_code IS NOT NULL AND project_id IS NULL AND can_dispose=FALSE) OR (scope_type='PROJECT' AND project_id IS NOT NULL AND menu_code IS NULL)) |
| `rule_access_permission_grant_3` | 업무 검증 | `can_edit` | can_edit 또는 can_dispose가 TRUE이면 can_view도 TRUE여야 한다. Audit Trail 메뉴의 can_edit는 FALSE이다. |
| `rule_access_permission_grant_4` | 업무 검증 | - | 활성 권한은 개인+메뉴, 그룹+메뉴, 개인+프로젝트, 그룹+프로젝트의 각 조합별로 중복을 허용하지 않는다. |
| `uq_access_user_id_menu` | UNIQUE | `user_id, menu_code, is_active, scope_type` | UNIQUE (user_id, menu_code) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_id_project` | UNIQUE | `user_id, project_id, is_active, scope_type` | UNIQUE (user_id, project_id) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='PROJECT' |
| `uq_access_user_group_id_menu` | UNIQUE | `user_group_id, menu_code, is_active, scope_type` | UNIQUE (user_group_id, menu_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_group_id_project` | UNIQUE | `user_group_id, project_id, is_active, scope_type` | UNIQUE (user_group_id, project_id) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='PROJECT' |
| `ck_access_view_required` | CHECK | `can_edit, can_dispose, can_view` | CHECK ((NOT can_edit AND NOT can_dispose) OR can_view) |

#### 업무 규칙

개인 직접 부여와 활성 소속 그룹의 부여를 합산하여 화면의 유효 권한을 계산한다. 그룹 상속 결과를 사용자별 직접 권한으로 중복 저장하지 않는다.

최종 권한은 활성 계정인지 확인한 뒤 해당 메뉴·프로젝트 범위의 개인 직접 권한과 활성 그룹·구성원에게 부여된 권한을 기능별 OR로 합산한다. 미부여는 허용하지 않는다. 편집·폐기에는 해당 접근권한과 업무 역할을 모두 요구하고, 검토·승인은 현재 Workflow의 담당 또는 유효한 대체 배정까지 확인한다. 접근권한은 승인 역할을 자동 부여하지 않으며 업무 역할도 접근권한을 자동 부여하지 않는다.

관리 권한등급은 관리 기능 범위를 정하고, 전역 역할은 user_role 및 GLOBAL group_role, 프로젝트 역할은 project_member 및 해당 프로젝트의 PROJECT group_role에서 계산한다. 인벤토리 업무는 inventory_role_grant를 적용한다. 관리자도 전자서명의 본인 확인·담당 배정·작성/검토/승인자 분리·승인 원문 잠금 규칙을 우회하지 않는다. 권한 변경과 그룹 탈퇴·비활성화는 다음 요청부터 유효 권한에 반영하며 서버에서 최종 판정한다.

[↑ 맨 위로](#top)

---

<a id="table-inventory_role_grant"></a>
### 9. 인벤토리 업무 역할 (`inventory_role_grant`)

| 항목 | 정의 |
|---|---|
| 설명 | 개인·그룹의 인벤토리 작성자·검토자·승인자·폐기 권한자 선택값 |
| Primary Key | `grant_id` |
| 주요 참조(FK) | `user_id, user_group_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 78 | 인벤토리 역할 ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 79 | 개인 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 80 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | user_group.user_group_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 81 | 인벤토리 역할 | `role_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | AUTHOR=작성자, REVIEWER=검토자, APPROVER=승인자, DISPOSER=폐기 권한자 | `APPROVER` |
| 82 | 활성 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 활성 여부 값 | `TRUE` |
| 83 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 84 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 85 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 86 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_inventory_role_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_inventory_role_grant_2` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','DISPOSER')) |
| `rule_inventory_role_grant_3` | 업무 검증 | - | 활성 개인+역할, 활성 그룹+역할의 각 조합별로 중복을 허용하지 않는다. |
| `uq_inventory_user_id` | UNIQUE | `user_id, role_code, is_active` | UNIQUE (user_id, role_code) WHERE is_active=TRUE AND user_id IS NOT NULL |
| `uq_inventory_user_group_id` | UNIQUE | `user_group_id, role_code, is_active` | UNIQUE (user_group_id, role_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL |

#### 업무 규칙

계정의 일반 업무 역할과 별도로 화면에서 지정하는 인벤토리 권한이다. 개인 부여와 활성 그룹 상속을 합산한다.

[↑ 맨 위로](#top)

---

## Compliance

<a id="table-electronic_signature"></a>
### 10. 전자서명 (`electronic_signature`)

| 항목 | 정의 |
|---|---|
| 설명 | 상신·검토·승인·폐기·수행 확인·결재선 적용 및 종료 요청의 전자서명 기록 |
| Primary Key | `signature_id` |
| 주요 참조(FK) | `signer_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 87 | 전자서명 ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 전자서명 기록 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 88 | 서명자 ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 89 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 실제 대상 테이블명. 업무 항목 개정, 산출물 개정, 종료 요청을 구분. 인벤토리는 system_asset PK와 revision_number로 식별 | `requirement` |
| 90 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 서명이 적용된 레코드 PK값 | `00000000-0000-0000-0000-000000000001` |
| 91 | 서명 단계 | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | SUBMIT / REVIEW / APPROVE / REJECT / DISPOSE / CONFIG_APPLY / EXECUTE. 서명 의미와 대상 업무를 함께 기록 | `APPROVE` |
| 92 | 서명 목적 | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 상신/검토/승인/폐기/시험 수행 확인의 업무 의미. 종료 요청은 PROJECT_CLOSE_NORMAL 또는 PROJECT_CLOSE_FORCED | `문서 최종 승인` |
| 93 | 서명 타임스탬프 | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 전자서명 수행 타임스탬프 | `2026-08-26T16:30:00Z` |
| 94 | 대상 버전 | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 개정 표시 버전 또는 개정번호. 종료 요청 CLOSE-{request_version}, 일탈 조치 ACTION-{round_number}-{revision_number}, 완료보고 COMPLETE-{마지막 조치 회차}-{최신 재수행 PK}, 미수행 종료 CLOSE-{마지막 조치 회차}. 시험 수행은 회차·결과 개정을 함께 식별한다. | `v1.0` |
| 95 | 서명 대상 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | payload_schema_version을 포함한 signed_payload의 정규화 UTF-8 JSON 전체에 대한 SHA-256 소문자 64자리. 저장한 원문에서 동일하게 재계산 가능해야 한다. | `[SAMPLE_SHA256_HASH]` |
| 96 | 재인증 방식 | `authentication_method` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 전자서명 수행 시 서명자 본인 확인에 사용한 재인증 방식 | `PASSWORD` |
| 97 | 재인증 결과 | `authentication_result` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 전자서명 수행 시 재인증 처리 결과 | `SUCCESS` |
| 98 | 서명자 표시명 | `signer_name_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 서명 당시 화면에 표시한 사용자명 | `홍길동` |
| 99 | 서명자 역할 | `signer_role_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 서명 당시 작성자/검토자/승인자 등 역할 | `승인자` |
| 100 | 서명 원문 형식 버전 | `payload_schema_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 대상 유형·서명 목적별 포함 필드와 정규화 규칙을 식별하는 버전 | `SIGN-1` |
| 101 | 서명 원문 | `signed_payload` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | 서버가 확정한 {schema_version,target,baseline_id,body,children,source_refs,files,context}. 각 대상에 없는 영역은 빈 배열 또는 NULL로 포함한다. 비밀번호·인증 비밀은 제외한다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_electronic_signature_1` | 업무 검증 | - | target_table_name과 target_record_id는 해당 대상의 실제 PK·개정을 함께 가리켜야 한다. 여러 테이블에 대한 다형 참조이며 물리 FK가 아니다. |
| `rule_electronic_signature_2` | 업무 검증 | - | 서명자와 처리자가 일치해야 하며, 성공한 재인증 결과만 유효 서명으로 저장한다. |
| `rule_electronic_signature_3` | 업무 검증 | - | 서명 후 내용은 수정하지 않는다. 대상 수정은 새 개정과 새 서명으로 처리한다. |
| `ck_signature_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |

#### 업무 규칙

폐기·시험 수행 확인·결재선 적용은 해당 업무 기록의 서명 FK로 연결한다. 일반 결재 처리는 workflow와 approval_action을 사용한다.

종료 서명은 project_closure_request PK와 CLOSE-{request_version}으로 식별한다. 과거 서명 원문과 해시는 보존한다.

signed_payload는 서명자가 확인한 본문, 순서가 있는 하위 절차, 근거 대상의 테이블·PK·버전·내용 해시, 서명 시 이미 존재하는 원본 첨부파일의 file_id·content_hash·크기, 프로젝트 검증 대상 baseline_id를 포함한다. 승인 원문에서 나중에 생성한 PDF 출력 파일은 이 원본 첨부 목록에 소급 추가하지 않는다. context에는 서명 목적·처리 사유를 포함하며 완료보고는 시정·예방 조치와 완료보고 원문, 미수행 종료는 종료 사유, 폐기는 폐기 사유와 원래 승인 대상 식별자를 포함한다. source_refs는 정확한 개정 원문을 보존한 대상만 참조한다. 반려 후 재상신으로 변경된 원문도 각 서명의 signed_payload에 별도로 남긴다.

정규화는 객체 키를 문자 코드 순으로 정렬하고 공백 없는 JSON을 UTF-8로 직렬화한다. 업무 순서가 있는 배열은 순서를 유지하고 집합형 참조·파일 목록은 식별자 순으로 정렬한다. 날짜는 UTC 표준 문자열, UUID는 소문자, NULL은 명시적으로 유지한다. 수치와 문자열 변환 규칙은 payload_schema_version별 정의를 보존한다. 서버가 대상과 하위 데이터·파일 해시를 다시 읽고 검증한 후 서명 원문 저장, 상태 전환, 감사기록을 같은 트랜잭션으로 확정한다.

승인 상태·현재 개정 표시·updated_at·결재 진행 요약·이후의 폐기 여부·invalidated_at 등 관리 필드는 원래 업무 승인 body에서 제외한다. 관리 필드 변경도 감사기록에 남기며 원래 signed_payload와 content_hash는 갱신하지 않는다. 폐기·종료 등의 별도 서명에서는 해당 처리 사유를 context에 포함한다. 동일 대상·목적·버전의 최종 승인 원문은 다시 덮어쓰거나 재승인하지 않으며 업무 원문 변경 시 새 버전 또는 새 조치 개정을 생성한다.

[↑ 맨 위로](#top)

---

<a id="table-audit_trail"></a>
### 11. Audit Trail (`audit_trail`)

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
| 102 | 감사추적 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 감사추적 레코드 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 103 | 수행자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 행위에는 필수. 시스템 이벤트는 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 104 | 작업 유형 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, LOGOUT, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL, PROJECT_CLOSE, PROJECT_FORCE_CLOSE | `REPORT_GENERATE` |
| 105 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 대상 실제 테이블명. 대상 행이 없는 이벤트는 NULL | `requirement` |
| 106 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 대상 PK. 로그인·로그아웃 등 대상 행이 없는 이벤트는 NULL | `00000000-0000-0000-0000-000000000001` |
| 107 | 변경 전 데이터 | `old_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 변경 전후 업무 값. 비밀번호·토큰·password_hash 제외 | `{"status": "작성중"}` |
| 108 | 변경 후 데이터 | `new_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 변경 전후 업무 값. 비밀번호·토큰·password_hash 제외 | `{"status": "승인완료"}` |
| 109 | 변경 사유 | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | 21 CFR Part 11 데이터 변경 사유 | `요구사항 오탈자 수정 및 규정 항목 보완` |
| 110 | 접속 IP 주소 | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | Y | Y | 사용자 클라이언트 IP | `192.0.2.10` |
| 111 | 발생 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 감사추적 로그 생성 시각 (UTC) | `2026-08-26T16:30:00Z` |
| 112 | 수행 주체 유형 | `actor_type` | `varchar(20)` | N | N | - | Y | `'USER'` | N | Y | N | Y | 변경 수행 주체 유형. USER, SYSTEM, BATCH로 구분하며 USER인 경우 actor_id를 필수로 저장 | `USER` |
| 113 | 요청 ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 하나의 화면·API 요청에서 발생한 여러 감사추적 기록을 동일 요청으로 묶기 위한 식별자 | `00000000-0000-0000-0000-000000000001` |
| 114 | 세션 ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 변경 작업이 발생한 사용자 로그인 세션 식별자. 시스템·배치 처리 또는 세션이 없는 요청은 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 115 | 대상 문서 버전 | `target_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 표시 버전을 저장하며 일반 테이블은 NULL 허용. 프로젝트 종료 요청·종료 기록에는 전자서명과 동일한 CLOSE-n 버전을 저장한다 | `v1.0` |
| 116 | 대상 개정 순번 | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 숫자형 개정 순번을 저장하며 일반 테이블은 NULL 허용 | `1` |
| 117 | 요청 경로 | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 변경을 발생시킨 화면 또는 API 요청 경로. 시스템·배치 처리 등 경로가 없는 경우 NULL 허용 | `/api/fds/approve` |
| 118 | 접속 클라이언트 정보 | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | 변경 요청에 사용된 브라우저, 운영체제 또는 클라이언트 애플리케이션 정보 | `Sample-Client/1.0` |
| 119 | 메뉴 구분 | `menu_code` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / ACCOUNTS / VP / VA / QIA / URS / FDS_GROUP / FRA / DQ / IQ / OQ / PQ / VSR / DASHBOARD / AUTH | `URS` |
| 120 | 행위자 역할 | `actor_role_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | 감사 화면의 역할 필터 및 당시 역할 표시 | `작성자` |
| 121 | 행위자 표시명 | `actor_display_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | 감사 화면에 표시하는 당시 사용자명 | `홍길동` |
| 122 | 변경 항목 | `field_path` | `text` | N | N | - | N | - | N | N | N | Y | 필드별 변경 화면에서 표시할 컬럼/항목 경로. 전체 이벤트면 NULL | `requirement_text` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_audit_trail_1` | 업무 검증 | - | 대상 테이블과 ID는 함께 존재하거나 함께 NULL이다. |
| `rule_audit_trail_2` | 업무 검증 | `actor_id` | USER 행위자는 actor_id 필수, SYSTEM 행위자는 NULL 가능. |
| `rule_audit_trail_3` | 업무 검증 | - | 감사 기록은 추가 전용이며 화면에서 수정·삭제하지 않는다. |

[↑ 맨 위로](#top)

---

## File

<a id="table-file_asset"></a>
### 12. 파일 자산 (`file_asset`)

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
| 123 | 파일 ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 첨부파일 메타데이터 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 124 | 업로드자 ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 사용자 업로드 파일은 업로드자 ID 필수. 시스템·정기 배치 생성 파일은 NULL 허용 | `UUID` |
| 125 | 원본 파일명 | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 업로드 당시 파일명 | `sample_document.pdf` |
| 126 | 저장 파일경로 | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | 스토리지 저장 경로/S3 Key | `documents/2026/08/sample_001.pdf` |
| 127 | 파일 구분 | `file_category` | `varchar(30)` | N | N | - | Y | `'ATTACHMENT'` | N | Y | N | Y | 파일 업무 구분. ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 128 | 파일 용량 | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | 파일 크기 (Byte) | `1048576` |
| 129 | MIME 타입 | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 첨부 파일 형식. 미확인 시 NULL 허용 | `application/pdf` |
| 130 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T16:30:00Z` |
| 131 | 파일 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | 저장 완료한 파일 원본 바이트의 SHA-256 소문자 64자리. 서버가 계산·검증한다. | - |
| 132 | 저장소 객체 버전 | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | 버전형 저장소의 정확한 객체 버전 ID. 버전 기능이 없으면 NULL이며 stored_file_path의 덮어쓰기를 금지한다. | - |
| 133 | 파일 생성 출처 | `file_origin` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | UPLOAD / SYSTEM_GENERATED. 서버가 실제 생성 경로로 지정하며 사용자 업로드는 검사 통과 및 검사 작업 연결 필수. file_category와 구분한다 | `UPLOAD` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_file_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes >= 0) |
| `ck_file_origin` | CHECK | `file_origin` | CHECK (file_origin IN ('UPLOAD','SYSTEM_GENERATED')) |
| `rule_file_asset_scan` | 업무 검증 | `file_origin, file_id` | UPLOAD는 위협 미검출 완료한 file_scan_job의 result_file_id와 연결하여 등록하고 해시·크기·업로드자·파일명·파일 구분의 일치를 검증한다. |
| `rule_file_asset_origin` | 업무 검증 | `file_origin` | SYSTEM_GENERATED는 서버 내부 생성 경로가 확인된 경우에만 허용하며 사용자 입력의 출처·파일 구분으로 검사 생략을 결정하지 않는다. |

#### 업무 규칙

VA·FDS·DDS 첨부 및 시험 증적은 파일 ID로 연결한다. 파일 교체 이력은 연결 대상의 개정/수행 회차에 보존한다.

승인 당시 첨부는 다른 파일로 덮어쓰지 않는다.

파일 저장과 서버 해시 검증이 완료된 경우에만 사용 가능한 file_asset을 생성한다. 사용자 업로드는 file_origin=UPLOAD로 기록하며, 추가로 file_scan_job의 최신 회차가 COMPLETED 및 NO_THREATS_FOUND이고 검사한 원본과 저장 파일의 바이트가 같아야 한다. 파일 자산 생성과 검사 작업의 result_file_id 연결은 같은 트랜잭션으로 확정한다. 검사 대기·진행·실패·검사 불가·위협 발견 파일은 이 테이블에 사용 가능한 파일로 등록하지 않는다.

file_origin은 서버가 실제 파일 생성 경로를 확인하여 지정한다. 신뢰된 서버 내부 처리로 생성한 PDF·내보내기 파일은 SYSTEM_GENERATED로 기록하고 사용자 업로드 검사 작업 없이 등록할 수 있다. 생성에 사용하는 외부 업로드 파일은 먼저 검사를 통과해야 한다. 사용자 업로드 파일은 file_category가 REPORT 또는 EXPORT여도 UPLOAD이며 검사 대상이다.

메타데이터만 존재하거나 검사·등록이 완료되지 않은 업로드는 업무 첨부, 다운로드·미리보기, 문서 파싱·AI 입력, 증적·승인·보고서 완료에 사용하지 않는다. VA·FDS·DDS, 규정 원문, 증적 및 문서의 파일 참조는 이 등록 조건을 충족한 file_id만 연결한다.

교체는 새 file_id와 새 저장 객체로 처리한다. 보존 기간에는 원본 바이트와 경로·버전·해시를 유지하고 객체 덮어쓰기·삭제를 차단한다. 저장소 버전이 있으면 조회 시 그 버전을 지정한다. 같은 내용의 서로 다른 첨부는 허용하므로 content_hash는 UNIQUE가 아니다. 파일 다운로드는 연결된 업무 대상의 조회 권한을 검사하며 업로드자라는 이유만으로 다른 업무 자료의 접근권한을 부여하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-file_scan_job"></a>
### 13. 파일 악성코드 검사 작업 (`file_scan_job`)

| 항목 | 정의 |
|---|---|
| 설명 | 사용자 업로드 파일의 격리 저장 정보, 악성코드 검사 회차·결과 및 검사 통과 후 파일 자산 등록 연결 |
| Primary Key | `scan_job_id` |
| 주요 참조(FK) | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 검사 회차·판정·파일 등록 이력은 감사 가능 기간 보존. 격리 원본은 검사 결과와 별도의 격리 파일 보존·삭제 정책에 따라 관리 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 134 | 검사 작업 ID | `scan_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 검사 한 회차의 고유 식별자. 큐 전달·결과 수신의 중복 처리 방지 기준 | `UUID` |
| 135 | 업로드 ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 서버가 발급한 한 업로드의 논리 식별자. 같은 원본의 재시도 회차는 동일 값을 유지하며 별도 테이블 FK가 아니다 | `UUID` |
| 136 | 검사 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 최초 검사는 1. 재시도·재검사는 같은 upload_id의 다음 회차로 추가하며 메시지 중복 수신만으로 증가시키지 않는다 | `1` |
| 137 | 이전 검사 작업 ID | `previous_scan_job_id` | `uuid` | N | Y | `file_scan_job.scan_job_id` | N | - | Y | Y | N | Y | 재시도·재검사의 직전 회차. 최초 회차는 NULL이며 직전 작업에서 여러 후속 회차가 갈라지지 않는다 | `UUID` |
| 138 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 첨부이면 해당 프로젝트. 규정 원문 등 프로젝트 외 업로드는 NULL을 허용하며 별도 메뉴·자료 접근권한을 검증한다 | `UUID` |
| 139 | 업로드자 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 원본 업로드를 수행한 사용자. 자동 재시도에서도 최초 업로드자를 유지한다 | `UUID` |
| 140 | 원본 파일명 | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 업로드 당시 파일명. 파일 자산 등록 시 동일 값을 사용한다 | `design_document.pdf` |
| 141 | 격리 저장소명 | `storage_bucket` | `varchar(255)` | N | N | - | Y | - | N | Y | N | Y | 검사 원본이 저장된 버킷 또는 저장소 식별자. 접속 비밀정보와 서명 URL은 저장하지 않는다 | `upload-quarantine-example` |
| 142 | 격리 파일경로 | `stored_file_path` | `text` | N | N | - | Y | - | N | Y | N | Y | 해당 업로드에 고유한 격리 객체 경로. 업무용 다운로드 경로로 노출하지 않는다 | `uploads/sample-upload/design_document.pdf` |
| 143 | 격리 객체 버전 | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | 검사한 정확한 객체 버전. 버전 미지원 저장소는 NULL이며 원본 경로의 덮어쓰기를 금지한다 | - |
| 144 | 원본 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | 서버 또는 신뢰된 검사 처리에서 확인한 원본 바이트의 SHA-256 소문자 64자리. 확인 전 NULL이며 위협 미검출 완료 시 필수 | - |
| 145 | 파일 용량 | `file_size_bytes` | `bigint` | N | N | - | Y | - | N | N | N | Y | 저장 완료한 원본의 실제 바이트 크기 | `1048576` |
| 146 | MIME 타입 | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 확인한 원본 파일 형식. 사용자 입력값만으로 파일 유형을 신뢰하지 않는다 | `application/pdf` |
| 147 | 파일 구분 | `file_category` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ATTACHMENT / EVIDENCE / REPORT / EXPORT. 업무 목적 구분이며 검사 생략 근거로 사용하지 않는다 | `ATTACHMENT` |
| 148 | 작업 상태 | `job_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED. COMPLETED는 검사가 종료되었다는 뜻이며 사용 허용 여부는 scan_result로 판단한다 | `PENDING` |
| 149 | 검사 결과 | `scan_result` | `varchar(30)` | N | N | - | N | - | N | Y | N | Y | NO_THREATS_FOUND / THREATS_FOUND / UNSCANNABLE. 위협 미검출·위협 발견·검사 불가를 구분하며 완료 전 또는 기술 실패·취소이면 NULL | `NO_THREATS_FOUND` |
| 150 | 검사 서비스·엔진명 | `scanner_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 해당 회차에 지정한 검사 서비스 또는 엔진의 식별자. 특정 제품에 종속하지 않는다 | `MALWARE_SCANNER` |
| 151 | 검사 엔진 버전 | `scanner_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 실제 사용한 엔진 버전. 검사 서비스가 제공하지 않으면 NULL을 허용한다 | - |
| 152 | 탐지 정보 버전 | `signature_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 검사에 사용한 악성코드 탐지 정보의 버전. 제공되지 않으면 NULL을 허용한다 | - |
| 153 | 검사 서비스 요청 ID | `scanner_request_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | 검사 서비스가 발급한 실행·요청 식별자. 결과를 작업·원본 객체와 대조할 때 사용한다 | - |
| 154 | 검사 상세 결과 | `scan_details` | `jsonb` | N | N | - | N | - | N | N | Y | Y | provider_result, reason_code, threat_names, report_reference 등 확인 가능한 검사 근거. 비밀번호·토큰·서명 URL·파일 본문은 제외한다 | `{"provider_result":"NO_THREATS_FOUND","threat_names":[]}` |
| 155 | 검사 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수동 검사·재검사 요청 사용자. 자동 처리이면 NULL을 허용하고 Audit Trail의 SYSTEM/BATCH 행위자로 기록한다 | `UUID` |
| 156 | 검사 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 해당 검사 회차를 등록한 시각 | `2026-09-18T01:00:00Z` |
| 157 | 검사 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 회차의 검사를 시작한 시각. 실행 전 실패·취소이면 NULL 가능 | `2026-09-18T01:00:05Z` |
| 158 | 검사 종료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 완료·실패·취소를 확정한 시각. 파일 자산 등록 시각과 구분한다 | `2026-09-18T01:00:20Z` |
| 159 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 작업 상태·검사 결과 또는 결과 파일 연결의 최종 변경 시각 | `2026-09-18T01:00:21Z` |
| 160 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FAILED의 기술 오류 코드. 위협 발견은 기술 오류가 아니며 scan_result로 기록한다 | `SCAN_TIMEOUT` |
| 161 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | Y | Y | 기술 실패 상세 사유. 비밀정보·서명 URL·파일 본문을 포함하지 않는다 | `검사 응답시간 초과` |
| 162 | 등록 파일 ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | 위협 미검출 후 원본 동일성을 확인하여 등록한 파일 자산. 등록 전에는 NULL이며 같은 업로드에서 한 회차만 연결한다 | `UUID` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_file_scan_attempt` | UNIQUE | `upload_id, attempt_no` | UNIQUE (upload_id, attempt_no) |
| `uq_file_scan_active` | UNIQUE | `upload_id` | UNIQUE (upload_id) WHERE job_status IN ('PENDING','PROCESSING') |
| `uq_file_scan_published` | UNIQUE | `upload_id` | UNIQUE (upload_id) WHERE result_file_id IS NOT NULL |
| `uq_file_scan_source` | UNIQUE | `storage_bucket, stored_file_path, storage_version_id` | UNIQUE (storage_bucket, stored_file_path, COALESCE(storage_version_id, '')) WHERE attempt_no=1 |
| `ck_file_scan_attempt` | CHECK | `attempt_no, previous_scan_job_id, scan_job_id` | CHECK ((attempt_no=1 AND previous_scan_job_id IS NULL) OR (attempt_no>1 AND previous_scan_job_id IS NOT NULL AND previous_scan_job_id<>scan_job_id)) |
| `ck_file_scan_status` | CHECK | `job_status` | CHECK (job_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |
| `ck_file_scan_result` | CHECK | `scan_result` | CHECK (scan_result IS NULL OR scan_result IN ('NO_THREATS_FOUND','THREATS_FOUND','UNSCANNABLE')) |
| `ck_file_scan_result_state` | CHECK | `job_status, scan_result, completed_at` | CHECK ((job_status IN ('PENDING','PROCESSING') AND scan_result IS NULL AND completed_at IS NULL) OR (job_status='COMPLETED' AND scan_result IS NOT NULL AND completed_at IS NOT NULL) OR (job_status IN ('FAILED','CANCELLED') AND scan_result IS NULL AND completed_at IS NOT NULL)) |
| `ck_file_scan_started` | CHECK | `job_status, started_at` | CHECK (job_status NOT IN ('PROCESSING','COMPLETED') OR started_at IS NOT NULL) |
| `ck_file_scan_time` | CHECK | `requested_at, started_at, completed_at` | CHECK ((started_at IS NULL OR started_at>=requested_at) AND (completed_at IS NULL OR completed_at>=requested_at) AND (started_at IS NULL OR completed_at IS NULL OR completed_at>=started_at)) |
| `ck_file_scan_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes>=0) |
| `ck_file_scan_hash` | CHECK | `content_hash` | CHECK (content_hash IS NULL OR content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_scan_clean_hash` | CHECK | `scan_result, content_hash` | CHECK (scan_result IS NULL OR scan_result<>'NO_THREATS_FOUND' OR content_hash IS NOT NULL) |
| `ck_file_scan_version` | CHECK | `storage_version_id` | CHECK (storage_version_id IS NULL OR length(storage_version_id)>0) |
| `ck_file_scan_category` | CHECK | `file_category` | CHECK (file_category IN ('ATTACHMENT','EVIDENCE','REPORT','EXPORT')) |
| `ck_file_scan_failed` | CHECK | `job_status, error_code` | CHECK (job_status<>'FAILED' OR error_code IS NOT NULL) |
| `ck_file_scan_publish` | CHECK | `result_file_id, job_status, scan_result, content_hash` | CHECK (result_file_id IS NULL OR (job_status='COMPLETED' AND scan_result IS NOT NULL AND scan_result='NO_THREATS_FOUND' AND content_hash IS NOT NULL)) |
| `rule_file_scan_chain` | 업무 검증 | `upload_id, attempt_no, previous_scan_job_id` | 이전 회차는 같은 upload_id의 attempt_no-1인 종료 행이어야 한다. 같은 업로드의 저장소·경로·버전·용량·업로드자·프로젝트·파일 구분을 유지하고 확인된 해시는 바꾸지 않는다. |
| `rule_file_scan_registration` | 업무 검증 | `result_file_id, content_hash` | 해당 업로드의 최신 회차가 위협 미검출이고 아직 등록되지 않은 경우에만 등록한다. file_asset의 file_origin=UPLOAD, 업로드자·파일명·구분·크기·해시가 검사 원본과 일치해야 한다. |
| `rule_file_scan_access` | 업무 검증 | `project_id, uploaded_by, requested_by` | 인증된 고객사에 속한 DB·저장소·사용자·프로젝트의 연결과 해당 업무의 파일 등록 권한을 검증한다. 요청에서 전달한 조직 ID·저장 경로만으로 고객사 DB나 접근권한을 결정하지 않는다. |

#### 업무 규칙

**검사 대상과 격리.** 사용자 업로드 파일은 저장 완료 후 이 테이블에 최초 검사 회차를 등록한다. 파일 유형·크기와 업로드 권한을 확인하고, 격리 원본은 검사 처리 주체에만 읽기를 허용한다. 검사 통과 및 파일 자산 등록 전에는 일반 사용자의 다운로드·미리보기, 업무 첨부, 문서 파싱·AI 입력, 승인·증적·보고서 완료에 사용할 수 없다. 사용자 업로드 파일의 file_category가 REPORT 또는 EXPORT여도 검사를 생략하지 않는다.

**작업과 판정.** 한 행은 한 검사 회차이며 PENDING에서 PROCESSING으로 전환한 뒤 COMPLETED 또는 FAILED로 종료한다. 시작 전 실패와 취소도 기록할 수 있다. COMPLETED의 NO_THREATS_FOUND만 파일 등록 후보이며, THREATS_FOUND와 UNSCANNABLE은 차단을 유지한다. 암호화·지원하지 않는 형식·검사 범위 제한 등으로 원본 전체를 검사하지 못하면 UNSCANNABLE로 기록하고 사유를 scan_details에 남긴다. 접근 오류·시간 초과·검사 서비스 장애는 FAILED로 기록한다. 알 수 없는 서비스 응답이나 결과 누락을 위협 미검출로 변환하지 않는다.

**재시도와 동시 처리.** 큐와 검사 서비스의 요청·결과에는 scan_job_id 및 검증된 고객사 연결 정보를 사용하고 원본 저장소·경로·객체 버전을 대조한다. 상태 전환은 현재 상태를 조건으로 갱신하며 중복 전달·중복 결과로 검사를 반복하거나 파일 자산을 중복 등록하지 않는다. 종료한 회차의 상태·원본·판정·검사 근거는 덮어쓰지 않는다. 기술 실패 후 다시 실행할 때는 같은 upload_id의 다음 회차를 생성한다. 검사 불가·위협 발견의 재검사는 권한 있는 요청과 사유를 기록하며 파일 내용이 바뀌면 새 upload_id로 등록한다. 이전 회차의 지연 응답으로 실패·취소를 성공으로 되돌리거나 최신 회차의 판정을 대체하지 않는다.

**작업 전달과 복구.** DB에 등록된 대기 작업은 큐 전송 실패 후에도 다시 전달할 수 있어야 한다. 제한 시간 내 결과가 없는 진행 작업은 실제 검사 실행을 확인한 후 실패 확정·다음 회차 등록으로 복구한다. 최대 시도 횟수와 실패 작업의 운영 확인 절차는 처리 정책으로 관리하며 무제한 재시도를 허용하지 않는다. 서비스 내부 재시도가 개별 회차로 제공되지 않는 경우에는 확인 가능한 요청 식별자·최종 결과만 기록하고 수신하지 않은 실행 이력을 생성하지 않는다.

**검사 원본과 등록 파일.** 검사 결과는 지정한 객체 버전 또는 덮어쓰기를 금지한 원본에만 유효하다. 위협 미검출 완료 후 업무 저장소로 복사하는 경우 최종 저장 객체의 해시·크기가 검사 원본과 같은지 확인하고 최종 경로·버전을 file_asset에 기록한다. 검사 이후 변환·편집한 파일을 같은 검사 원본으로 등록하지 않는다. 최초 검사 대상의 위치·버전은 이 테이블에서 유지한다.

**파일 등록의 일관성.** file_asset 생성, 이 회차의 result_file_id 연결, 감사기록을 같은 DB 트랜잭션으로 확정한다. 외부 저장소 작업은 DB 트랜잭션과 별개이므로 최종 저장 확인 전에는 파일을 사용 가능하게 노출하지 않는다. 같은 업로드의 최초 회차를 잠그거나 동등한 동시 처리 제어를 적용해 다음 회차 생성과 파일 등록이 동시에 확정되지 않도록 한다. 위협 미검출이어도 파일 등록이 끝나지 않았다면 result_file_id는 NULL이며 사용을 허용하지 않는다. 저장·등록 실패 시 원본 동일성을 다시 확인하여 같은 회차의 등록을 재개할 수 있고, 이때 종료된 검사 결과는 바꾸지 않는다. 등록 완료한 업로드는 이 테이블에서 추가 검사 회차를 생성하지 않는다.

**권한과 결과 신뢰.** Worker나 결과 수신 처리는 서버가 관리하는 고객사 접속 정보로 해당 DB와 저장소를 선택한다. 검사 결과의 송신 주체·실행 식별·원본 객체를 검증하고 사용자 입력으로 판정을 확정하지 않는다. 검사 통과는 파일 접근권한이나 업무 승인 완료를 의미하지 않으며 실제 연결·다운로드 시 기존 업무 권한을 다시 검사한다.

**이력과 보존.** 검사 요청·상태 전환·최종 판정·파일 등록은 Audit Trail에 기록한다. 보존 대상 파일에 연결한 검사 결과는 해당 파일의 감사 가능 기간 동안 유지한다. 미등록·악성·검사 불가 원본의 삭제 여부와 기간은 격리 파일 정책으로 관리하고, 원본을 정리하더라도 검사 결과와 처리 사유를 보존한다. 격리 원본에 승인 증적의 장기 잠금 정책을 일괄 적용하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-evidence_link"></a>
### 14. 증적 파일 연결 (`evidence_link`)

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
| 163 | 증적 연결 ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서·시험 항목과 증적 파일 간 연결 고유 식별자. deleted_at IS NULL인 행에 (project_id, file_id, target_entity_type, target_entity_id, evidence_type) 중복을 허용하지 않는다. | `UUID` |
| 164 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 증적 연결이 속한 프로젝트 ID | `UUID` |
| 165 | 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | 연결되는 증적 파일 ID | `UUID` |
| 166 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FDS_SPEC / DDS_SPEC / VP_PLAN / QIA_MODULE_ITEM / VENDOR_AUDIT / DQ_ITEM / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / IQ_EXECUTION / OQ_EXECUTION / PQ_EXECUTION / IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION / VSR_ASSESSMENT / DEVIATION / DELIVERABLE_REVISION | `IQ_STEP_EXECUTION` |
| 167 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | target_entity_type에 해당하는 테이블의 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `UUID` |
| 168 | 증적 유형 | `evidence_type` | `varchar(50)` | N | N | - | Y | `'TEST_RESULT'` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 169 | 증적 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 증적 파일의 내용 및 연결 목적 | `IQ 수행 결과 화면 캡처` |
| 170 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 171 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 생성한 사용자 ID | `UUID` |
| 172 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 173 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 최종 수정한 사용자 ID | `UUID` |
| 174 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 증적 연결 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_evidence_link_1` | UNIQUE | `project_id, file_id, target_entity_type, target_entity_id, evidence_type, deleted_at` | UNIQUE (project_id,file_id,target_entity_type,target_entity_id,evidence_type) WHERE deleted_at IS NULL |
| `rule_evidence_link_2` | 업무 검증 | - | 대상 유형별 실제 PK·동일 프로젝트·정확한 개정을 검사한다. 다형 참조를 물리 FK로 표시하지 않는다. |
| `rule_evidence_link_3` | 업무 검증 | - | 절차별 첨부는 각 IQ/OQ/PQ_STEP_EXECUTION에, 회차 전체 첨부는 각 EXECUTION에 연결한다. 승인된 대상의 첨부는 덮어쓰지 않는다. |
| `rule_evidence_link_4` | 업무 검증 | `file_id` | file_asset의 등록 조건을 충족한 파일만 연결한다. UPLOAD이면 위협 미검출 검사와 등록 파일의 원본 동일성을 확인하며 격리 경로·검사 작업 ID를 file_id 대신 사용하지 않는다. |

#### 업무 규칙

VA/FDS/DDS의 대표 첨부는 해당 테이블의 file_id가 원본이다. 동일 대표 첨부를 여기서 별도 원본으로 중복 편집하지 않는다.

[↑ 맨 위로](#top)

---

## System

<a id="table-system_asset"></a>
### 15. 시스템/장비 식별 정보 (`system_asset`)

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
| 175 | 시스템 ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 시스템 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 176 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | 시스템/장비가 소속된 고객사 또는 운영 조직 | `00000000-0000-0000-0000-000000000001` |
| 177 | 관리번호 | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 자산/설비 관리번호 | `EQ-MES-2024-001` |
| 178 | 시스템명 | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 시스템/장비명 | `Sample System` |
| 179 | 대분류 | `major_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 컴퓨터화 시스템/장비/시설·유틸리티 | `컴퓨터화 시스템` |
| 180 | 중분류 | `middle_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 선택 대분류에 속하는 중분류 | `품질 시스템` |
| 181 | 대상 | `target_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 선택 대분류·중분류에 속하는 대상 | `LIMS` |
| 182 | 담당부서 | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 관리/운용 담당부서 | `생산기술팀` |
| 183 | 설치위치 | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 물리적/논리적 설치 장소 | `서버실 A동 3F` |
| 184 | 공급업체 | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 장비/시스템 공급업체명 | `Sample Vendor` |
| 185 | 모델명 | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 장비/시스템 모델명 | `FillMaster 500` |
| 186 | 시스템 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 시스템 목적 및 운영 범위 설명 | `생산 공정 데이터 수집 및 제어` |
| 187 | CS 포함 여부 | `is_cs_included` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Computerized System 포함 여부 | `TRUE` |
| 188 | 소프트웨어 버전 | `software_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 컴퓨터화 시스템의 소프트웨어 버전. 인벤토리 개정번호와 구분한다. | `4.8.2` |
| 189 | 인벤토리 개정번호 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 개정 기능으로 증가하는 정수 버전. 소프트웨어 버전과 별도 | `2` |
| 190 | GAMP 범주 | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 화면 선택값 CATEGORY_1/CATEGORY_2/CATEGORY_3/CATEGORY_4. 미선택은 NULL | `CATEGORY_4` |
| 191 | GxP 대상 여부 | `gxp_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=대상, NOT_APPLICABLE=비대상. 미선택은 NULL | `APPLICABLE` |
| 192 | Part 11 대상 여부 | `part11_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=대상, NOT_APPLICABLE=비대상. 미선택은 NULL | `APPLICABLE` |
| 193 | 승인 상태 | `approval_status` | `varchar(30)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=작성중, REVIEW=검토중, APPROVAL=승인중, APPROVED=승인 완료, REAPPROVAL_REQUIRED=재승인 필요, REJECTED=반려 | `DRAFT` |
| 194 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 195 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 196 | 사용·폐기 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE=사용, DISPOSED=폐기. 승인 상태와 별도 | `ACTIVE` |
| 197 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 전자서명 완료 시각 | `2026-09-01T00:00:00Z` |
| 198 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 화면에서 입력하는 사유 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_system_asset_1` | UNIQUE | `organization_id, management_number` | UNIQUE (organization_id, management_number) |
| `ck_system_asset_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_system_asset_3` | CHECK | `gxp_applicability` | CHECK (gxp_applicability IS NULL OR gxp_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_4` | CHECK | - | CHECK (part11_applicability IS NULL OR part11_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |

#### 업무 규칙

컴퓨터화 여부에 따라 소프트웨어 버전·GAMP·GxP·Part 11의 화면 필수값을 검사한다. 분류 3단계는 선택 가능한 조합만 허용한다.

연결 프로젝트 목록은 validation_project.system_id로 조회한다. 개정·승인·폐기 이력은 system_asset_revision 및 전자서명/감사기록으로 연결한다.

승인 완료된 정보를 변경할 때 개정번호를 증가시키고 재승인 상태를 표시한다. 소프트웨어 버전 자체의 변경과 인벤토리 문서 개정을 혼동하지 않는다.

승인된 업무 필드의 수정은 새 revision_number와 재승인으로 처리한다. 최신 시스템 정보의 변경은 과거 system_asset_revision 원문이나 프로젝트가 채택한 승인 개정을 바꾸지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-system_asset_revision"></a>
### 16. 시스템 인벤토리 개정 이력 (`system_asset_revision`)

| 항목 | 정의 |
|---|---|
| 설명 | 인벤토리의 개정번호별 등록·수정·승인·폐기 처리 이력 |
| Primary Key | `system_revision_id` |
| 주요 참조(FK) | `system_id, processed_by, signature_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 199 | 인벤토리 이력 ID | `system_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 200 | 시스템 ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | system_asset.system_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 201 | 개정번호 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 처리 당시 인벤토리 개정번호 | `2` |
| 202 | 처리 구분 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | REGISTER=등록, SAVE=저장, REVISE=개정, APPROVE=승인, DISPOSE=폐기 | `REVISE` |
| 203 | 변경·처리 사유 | `change_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 또는 폐기 시 필수인 사유 | - |
| 204 | 처리 후 승인 상태 | `approval_status` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | system_asset.approval_status와 같은 코드 | `REVIEW` |
| 205 | 처리자 ID | `processed_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 206 | 처리 시각 | `processed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이력 화면의 일시 | `2026-09-01T00:00:00Z` |
| 207 | 처리 전자서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 승인·폐기에 연결된 전자서명. 복수 서명은 동일 대상의 서명 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 208 | 원문 형식 버전 | `snapshot_schema_version` | `varchar(50)` | N | N | - | Y | `'ASSET-1'` | N | N | N | Y | asset_snapshot의 필드 구성·자료형 버전. 과거 형식 정의를 보존한다. | `ASSET-1` |
| 209 | 처리 당시 인벤토리 원문 | `asset_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 처리 후 system_asset의 모든 컬럼 값을 필드명·NULL을 포함하여 저장한다. 승인 이벤트는 최종 승인한 업무 원문과 일치해야 한다. | - |
| 210 | 원문 해시 | `snapshot_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | {schema_version:snapshot_schema_version,body:asset_snapshot}을 전자서명과 같은 정규화 규칙으로 직렬화한 SHA-256 소문자 64자리. 전자서명의 전체 서명 묶음 해시와 구분한다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_system_asset_revision_1` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `rule_system_asset_revision_2` | 업무 검증 | - | 동일 시스템·개정번호에 등록/저장/승인 등 여러 처리 이력이 존재할 수 있으므로 해당 두 컬럼만의 UNIQUE는 두지 않는다. |
| `rule_system_asset_revision_3` | 업무 검증 | - | action_type이 REVISE 또는 DISPOSE이면 change_reason은 비어 있을 수 없다. |
| `rule_system_asset_revision_4` | 업무 검증 | - | DISPOSE 처리는 signature_id가 필수이며 signature_action=DISPOSE, 서명 대상은 system_asset의 system_id와 해당 revision_number이다. |
| `uq_asset_approved_revision` | UNIQUE | `system_id, revision_number, action_type` | UNIQUE (system_id, revision_number) WHERE action_type='APPROVE' |
| `ck_asset_event_kind` | CHECK | `action_type` | CHECK (action_type IN ('REGISTER','SAVE','REVISE','APPROVE','DISPOSE')) |
| `ck_asset_approval_signature` | CHECK | `action_type, approval_status, signature_id` | CHECK (action_type <> 'APPROVE' OR (approval_status='APPROVED' AND signature_id IS NOT NULL)) |

#### 업무 규칙

각 처리 이벤트와 당시 전체 원문을 함께 추가 저장한다. 이벤트 행과 원문은 수정·삭제하지 않는다. 변경 전후 비교는 audit_trail에서 조회하고, 과거 승인 원문은 APPROVE 이벤트의 asset_snapshot에서 직접 조회한다. 승인 대상은 system_asset의 system_id와 해당 revision_number로 식별한다.

APPROVE 이벤트는 검토 단계별 이벤트가 아닌 최종 승인 완료 기록이다. 서명 대상 system_id·개정번호와 승인 원문의 업무 필드가 일치해야 한다. 승인 상태·처리 시각 등 관리 메타데이터를 제외한 비교 범위는 전자서명 원문 형식에 따른다. 동일 개정의 최종 승인 원문은 한 건이며 승인 후 업무 내용 변경에는 새 개정번호를 부여한다. 저장·개정·서명·최종 승인 이벤트는 원본 변경과 같은 트랜잭션에서 기록한다.

[↑ 맨 위로](#top)

---

## Library

<a id="table-library_item"></a>
### 17. 라이브러리 항목 마스터 (`library_item`)

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
| 211 | 라이브러리 ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 라이브러리 항목 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 212 | 모듈 구분 | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | URS/FRA/IQ/OQ/PQ 5개 라이브러리 탭 구분 | `URS` |
| 213 | 코드 | `code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 라이브러리 종류 내 항목 코드. (module_type, code) 복합 UNIQUE 적용 | `URS-AT-L01` |
| 214 | 분류/대분류 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | URS에서는 대분류, FRA/IQ/OQ/PQ에서는 분류를 저장한다. | `감사추적` |
| 215 | 항목명/기능명 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | URS에서는 기능명, FRA에서는 위험 항목명, IQ/OQ/PQ에서는 시험 항목명 | `데이터 변경 감사추적 자동 생성` |
| 216 | 요구사항 / 절차 | `requirement_text` | `text` | N | N | - | N | - | N | N | N | Y | URS 요구사항 본문. URS 종류에서 필수이며 다른 종류는 NULL 허용 | `모든 데이터 생성·수정·삭제 시...` |
| 217 | 기대 결과 | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ 예상 결과. 시험 라이브러리에서 필수 | - |
| 218 | 수용 기준/적용범위 | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | URS에서는 적용범위, IQ/OQ/PQ에서는 수용 기준. FRA는 NULL 허용 | `데이터 변경 시 Audit Trail 자동 생성...` |
| 219 | 근거 규정 | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | 화면의 규정 근거 표시·수기 입력. 구조화한 조항은 library_item_regulation에 연결하며 같은 근거를 함께 편집하지 않는다. | `21 CFR 11.10(e)` |
| 220 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 활성 여부 (TRUE/FALSE) | `TRUE` |
| 221 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 222 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 223 | 제공 유형 | `provision_type` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | URS 전용. BUILTIN=기본 제공, CUSTOM=사용자 지정 | `CUSTOM` |
| 224 | 중분류 | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 전용 중분류 | - |
| 225 | 대상 | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 전용 적용 대상 | - |
| 226 | 시험 내용 | `test_content` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ 전용 시험 절차·내용 | - |
| 227 | 위험 시나리오 | `risk_scenario` | `text` | N | N | - | N | - | N | N | N | Y | FRA 전용 위험 시나리오 | - |
| 228 | 심각도(SEV) | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA 전용. 1~5 | - |
| 229 | 발생도(OCC) | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA 전용. 1~5 | - |
| 230 | 검출도(DET) | `detectability` | `char(1)` | N | N | - | N | - | N | N | N | Y | FRA 전용. H/M/L | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_library_item_1` | UNIQUE | `module_type, code` | UNIQUE (module_type, code) |
| `ck_library_item_2` | CHECK | `module_type` | CHECK (module_type IN ('URS','FRA','IQ','OQ','PQ')) |
| `rule_library_item_3` | 업무 검증 | - | URS: 제공 유형·대분류·중분류·대상·기능명·요구사항·적용범위 필수. FRA: 분류·위험 항목명·시나리오·SEV·OCC·DET 필수. IQ/OQ/PQ: 분류·항목명·시험 내용·예상 결과·수용 기준 필수. |
| `rule_library_item_4` | 업무 검증 | `severity` | FRA의 severity/occurrence는 1~5, detectability는 H/M/L 중 하나이다. |

#### 업무 규칙

URS의 대분류는 category, 기능명은 title, 적용범위는 acceptance_criteria에서 관리한다.

템플릿을 요구사항·위험·시험에 적용할 때 본문과 입력값을 복사한다. 이후 라이브러리 수정이 이미 작성·승인한 항목을 자동 변경하지 않는다.

[↑ 맨 위로](#top)

---

## Validation

<a id="table-validation_project"></a>
### 18. Validation 프로젝트 (`validation_project`)

| 항목 | 정의 |
|---|---|
| 설명 | 시스템별 밸리데이션 수행 단위 및 범위 (VP) |
| Primary Key | `project_id` |
| 주요 참조(FK) | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 231 | 프로젝트 ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 232 | 프로젝트 코드 | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | 프로젝트 식별 코드 | `VP-SYS-008-20260422` |
| 233 | 프로젝트명 | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트명 | `테스트 장비3 CSV 프로젝트` |
| 234 | 시스템 ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | system_asset.system_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 235 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS=진행중, CLOSED_NORMAL=정상종료, CLOSED_FORCED=강제종료 | `IN_PROGRESS` |
| 236 | 검증 방식 | `validation_type` | `varchar(50)` | N | N | - | Y | `'NEW'` | N | N | N | Y | NEW=신규, CHANGE=변경, REVALIDATION=재검증 | `NEW` |
| 237 | 밸리데이션 레벨 | `validation_level` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | LEVEL_1/LEVEL_2/LEVEL_3/CUSTOM. CUSTOM=사용자 지정이며 수행 활동은 project_activity로 저장 | `LEVEL_2` |
| 238 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 추가 메모 사항 | - |
| 239 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 240 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 생성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 241 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 242 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 243 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 244 | 종료 유형 | `closure_type` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 유형 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `FORCED` |
| 245 | 강제종료 사유 | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | 현재/최종 project_closure_request의 강제종료 사유 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `사업 우선순위 변경으로 Validation 중단` |
| 246 | 종료 요청자 ID | `closure_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 요청자 ID 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 247 | 종료 요청 시각 | `closure_requested_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 요청 시각 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `2026-09-14T09:00:00Z` |
| 248 | 종료 완료 시각 | `closed_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 완료 시각 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `2026-09-14T09:05:00Z` |
| 249 | 현재 종료 요청 ID | `current_closure_request_id` | `uuid` | N | Y | `project_closure_request.closure_request_id` | N | - | N | Y | N | Y | 종료 진행 상태 및 사유를 표시할 종료 요청 | `00000000-0000-0000-0000-000000000001` |
| 250 | 현재 검증 대상 기준 | `current_system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 현재 is_current=TRUE인 동일 프로젝트의 기준. 최초 업무 상신 또는 시험 수행 전에 필수 | - |
| 251 | 적용 선후행 조건 버전 | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 생성 시 채택한 선후행 규칙 묶음 버전 | `DEPENDENCY-1` |
| 252 | 적용 선후행 조건 원문 | `dependency_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 생성 당시 활성 조건 전체와 판정 의미를 보존. {schema_version,version,definitions,conditions:[{activity_dependency_id,successor_activity_id,predecessor_activity_id,dependency_type,required_status,condition_type,condition_value,evaluation_order}]} | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_validation_project_1` | CHECK | `status` | CHECK (status IN ('IN_PROGRESS','CLOSED_NORMAL','CLOSED_FORCED')) |
| `ck_validation_project_2` | CHECK | `validation_type` | CHECK (validation_type IN ('NEW','CHANGE','REVALIDATION')) |
| `ck_validation_project_3` | CHECK | `validation_level` | CHECK (validation_level IN ('LEVEL_1','LEVEL_2','LEVEL_3','CUSTOM')) |
| `rule_project_current_refs` | 업무 검증 | `current_closure_request_id, current_system_baseline_id` | 현재 종료 요청과 검증 대상 기준은 모두 이 프로젝트에 속해야 한다. 기준이 존재하면 현재 기준 참조는 is_current=TRUE인 유일한 행과 일치해야 한다. |

#### 업무 규칙

화면의 프로젝트명·대상 시스템·검증 유형·수준·메모는 이 테이블에서 관리한다. 진행률은 선택 수행 활동의 완료 상태로 계산하고 RTM은 분모에서 제외한다.

프로젝트의 검증 대상 GAMP·GxP·소프트웨어 버전은 current_system_baseline_id가 가리키는 승인 원문에서 조회한다. 시스템 최신 정보와 검증 대상 기준을 구분하여 표시한다.

종료 요청의 검토/승인 진행은 project_closure_request 및 workflow_instance에 저장한다. 최종 승인 후에만 프로젝트를 정상종료 또는 강제종료로 변경한다.

프로젝트 생성 시 dependency_set_version과 dependency_snapshot을 함께 고정하며, 전역 activity_dependency 변경을 진행 중·종료 프로젝트에 자동 전파하지 않는다. 조건의 정의에는 상태별 완료 판정, AND/OR 결합, 미선택 활동 처리 및 위험 커버리지 판정 기준을 포함한다. 프로젝트 평가에는 이 사본을 사용하고 생성 후 직접 교체하지 않는다. 업무 승인·시험 수행 전에 유효한 current_system_baseline_id를 지정하며, 최초 기준 등록과 현재 참조 설정은 한 트랜잭션으로 처리한다.

[↑ 맨 위로](#top)

---

<a id="table-project_system_baseline"></a>
### 19. 프로젝트 검증 대상 개정 (`project_system_baseline`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트가 검증 대상으로 채택한 인벤토리 승인 개정과 변경 이력 |
| Primary Key | `baseline_id` |
| 주요 참조(FK) | `project_id, system_revision_id, adopted_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존. 승인 원문 및 과거 연결 이력은 삭제하지 않는다. |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 253 | 검증 대상 기준 ID | `baseline_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트가 채택한 한 번의 검증 대상 기준 식별자 | - |
| 254 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 검증 대상 기준이 속하는 프로젝트 | - |
| 255 | 인벤토리 승인 이력 ID | `system_revision_id` | `uuid` | N | Y | `system_asset_revision.system_revision_id` | Y | - | N | Y | N | Y | action_type=APPROVE인 변경 불가능한 원문 이벤트 | - |
| 256 | 기준 순번 | `baseline_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 프로젝트별 1부터 증가하며 변경 시 새 행을 생성한다. | `1` |
| 257 | 채택·변경 사유 | `change_reason` | `text` | N | N | - | Y | - | N | N | N | Y | 최초 대상 선정 또는 승인 개정 변경의 사유. 공백 불가 | - |
| 258 | 현재 적용 여부 | `is_current` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 현재 프로젝트 기준 여부. 과거 행은 FALSE로 변경하고 원문 연결은 유지한다. | - |
| 259 | 채택 시각 | `adopted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 서버에서 기록한 적용 시각 | - |
| 260 | 채택자 | `adopted_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 대상 설정 권한이 있는 처리자 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_baseline_number` | UNIQUE | `project_id, baseline_number` | UNIQUE (project_id, baseline_number) |
| `uq_project_baseline_current` | UNIQUE | `project_id, is_current` | UNIQUE (project_id) WHERE is_current=TRUE |
| `ck_project_baseline_number` | CHECK | `baseline_number` | CHECK (baseline_number >= 1) |
| `ck_project_baseline_reason` | CHECK | `change_reason` | CHECK (length(trim(change_reason)) > 0) |
| `rule_project_baseline_asset` | 업무 검증 | `project_id, system_revision_id` | 승인 이벤트의 system_id는 프로젝트의 system_id와 같아야 한다. 채택 시 시스템이 사용 중이며 해당 개정이 현재 유효한 승인본인지 검증한다. 이전 기준의 과거 승인 이벤트가 존재한다는 이유만으로 이를 최신 기준으로 채택하지 않는다. |

#### 업무 규칙

채택한 system_revision_id·순번·사유·처리자·시각은 덮어쓰지 않는다. 변경 시 프로젝트를 잠그고 이전 is_current를 FALSE로 전환한 후 새 행과 현재 참조를 같은 트랜잭션으로 저장한다. 종료 프로젝트의 기준은 변경하지 않는다.

대상 개정 변경은 영향받는 활동·문서의 재검토 및 재승인 상태를 갱신하며, 이미 승인한 기록의 기준 연결은 유지한다. 과거 시험 수행과 문서는 각자의 baseline_id를 통해 당시 소프트웨어 버전·분류·GxP 여부를 조회한다.

[↑ 맨 위로](#top)

---

<a id="table-project_member"></a>
### 20. 프로젝트 참여자 (`project_member`)

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
| 261 | 프로젝트 참여자 ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 참여자 역할 매핑 고유 식별자. deleted_at IS NULL이고 member_status가 ACTIVE인 행에 (project_id, user_id, role_id) 중복을 허용하지 않는다. | `UUID` |
| 262 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 참여자가 소속된 Validation 프로젝트 ID | `UUID` |
| 263 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트에 참여하는 사용자 ID | `UUID` |
| 264 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | 프로젝트 내에서 사용자가 수행하는 역할 ID | `UUID` |
| 265 | 참여 상태 | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | 프로젝트 참여 상태. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 266 | 참여 시작 시각 | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여가 시작된 시각 | `2026-09-01T10:00:00Z` |
| 267 | 참여 종료 시각 | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여가 종료된 시각. 현재 참여 중이면 NULL | `2026-09-01T00:00:00Z` |
| 268 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 269 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 등록한 사용자 ID | `UUID` |
| 270 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 271 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 최종 수정한 사용자 ID | `UUID` |
| 272 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여 정보 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_member_1` | UNIQUE | `project_id, user_id, role_id, deleted_at, member_status` | UNIQUE (project_id, user_id, role_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |

#### 업무 규칙

프로젝트별 참여자와 역할을 관리한다. 화면의 조회/편집/폐기 권한은 access_permission_grant에서 별도로 관리한다.

[↑ 맨 위로](#top)

---

<a id="table-validation_activity"></a>
### 21. 밸리데이션 활동 마스터 (`validation_activity`)

| 항목 | 정의 |
|---|---|
| 설명 | VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, VSR 수행 활동 기준정보 관리 |
| Primary Key | `activity_id` |
| 주요 참조(FK) | `created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 273 | 활동 ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 밸리데이션 활동 고유 식별자 | `UUID` |
| 274 | 활동 코드 | `activity_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, VSR. FDS_GROUP은 화면의 F&DS 표시명에 대응 | `FDS_GROUP` |
| 275 | 활동명 | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 화면 표시용 활동명 | `사용자 요구사항 명세` |
| 276 | 활동 순서 | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 기본 표시 순서: VP→VA→QIA→URS→F&DS→FRA→DQ→IQ→OQ→PQ→VSR. F&DS 내부 문서는 FDS 다음 DDS | `5` |
| 277 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 활동 마스터 사용 여부 | `TRUE` |
| 278 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 279 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 등록한 사용자 | `UUID` |
| 280 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 281 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 최종 수정한 사용자 | `UUID` |
| 282 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 마스터 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 업무 규칙

RTM은 요구사항·설계·위험·시험의 추적 관계를 조회하는 대시보드 기능이다. 시스템 인벤토리는 프로젝트의 선행 기준 정보로 관리한다.

FDS와 DDS는 F&DS 한 활동에 속하는 문서 종류다. 두 종류를 각각 별도의 수행 단계로 중복 생성하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-project_activity"></a>
### 22. 프로젝트 수행 활동 (`project_activity`)

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
| 283 | 프로젝트 활동 ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 수행 활동 고유 식별자. deleted_at IS NULL인 행에 (project_id, activity_id) 중복을 허용하지 않는다. 수행 대상 선택 여부와 무관하게 적용한다. | `UUID` |
| 284 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 활동이 속한 Validation 프로젝트 | `UUID` |
| 285 | 활동 ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 프로젝트에서 수행할 활동 | `UUID` |
| 286 | 수행 대상 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 프로젝트 설정에서 선택한 실제 수행 활동인지 여부. RTM 선택행을 만들지 않는다. | `TRUE` |
| 287 | 필수 활동 여부 | `is_required` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 해당 프로젝트에서 생략할 수 없는 활동인지 여부 | `TRUE` |
| 288 | 활동 상태 | `activity_status` | `varchar(20)` | N | N | - | Y | `'LOCKED'` | N | Y | N | Y | 활동 상태. LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED. 현재 Revision 기준으로 집계하며 산출물 변경·개정 시 재평가한다. COMPLETED/APPROVED는 단순 화면 입력값으로 변경하지 않는다 | `READY` |
| 289 | 활성화 시각 | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 선행 조건 충족으로 활동이 READY가 된 시각 | `2026-09-01T10:00:00Z` |
| 290 | 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 시작 시각 | `2026-09-01T11:00:00Z` |
| 291 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 완료 시각 | `2026-09-02T15:00:00Z` |
| 292 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동의 최종 승인 완료 시각 | `2026-09-02T17:00:00Z` |
| 293 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 294 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 등록 사용자 | `UUID` |
| 295 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 296 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 최종 수정 사용자 | `UUID` |
| 297 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 활동 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_activity_1` | UNIQUE | `project_id, activity_id, deleted_at` | UNIQUE (project_id, activity_id) WHERE deleted_at IS NULL |

#### 업무 규칙

진행률·완료 활동 수·정상종료 조건은 선택된 실제 수행 활동을 대상으로 한다. RTM 자체 승인·산출물은 요구하지 않는다. FDS/DDS의 진행은 하나의 F&DS 활동에 집계한다.

[↑ 맨 위로](#top)

---

<a id="table-activity_dependency"></a>
### 23. 활동 선후행 조건 (`activity_dependency`)

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
| 298 | 활동 선후행 조건 ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 활동 선후행 조건 고유 식별자 | `UUID` |
| 299 | 후행 활동 ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 조건 충족 후 활성화되는 활동 | `UUID` |
| 300 | 선행 활동 ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | 후행 활동 활성화 전에 확인할 활동. 전체 활동 조건이면 NULL 허용. STATUS 조건이면 필수이다 | `UUID` |
| 301 | 관계 구분 | `dependency_type` | `varchar(20)` | N | N | - | Y | `'REQUIRED'` | N | Y | N | Y | 선후행 관계 구분. REQUIRED, RECOMMENDED | `REQUIRED` |
| 302 | 요구 상태 | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | STATUS 조건의 판정 기준. CREATED는 유효한 현재 산출물 존재, COMPLETED는 수행 완료, APPROVED는 현재 산출물 최종 승인을 뜻한다. project_activity.activity_status에 CREATED를 저장하지 않는다. STATUS 조건에서만 필수이며 상세 판정은 아래 업무 규칙를 따른다. | `APPROVED` |
| 303 | 조건 유형 | `condition_type` | `varchar(50)` | N | N | - | Y | `'STATUS'` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED. 화면의 선행 단계 및 완료 조건을 판정하는 구분 | `STATUS` |
| 304 | 조건 값 | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 조건 판정 인자. STATUS + CREATED에서는 허용된 대상 산출물 테이블명(예: requirement)을 저장한다. 다른 조건의 인자는 해당 조건 구현에서 검증하며 불필요하면 NULL로 둔다. | - |
| 305 | 조건 설명 | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | 활동 활성화에 필요한 선행 활동과 승인 상태의 설명 | `URS 승인완료` |
| 306 | 평가 순서 | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 동일 후행 활동의 조건 평가 순서. 평가 순서는 조건 간 AND/OR 관계를 바꾸지 않는다 | `1` |
| 307 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 활성화 조건 사용 여부 | `TRUE` |
| 308 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 309 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 등록 사용자 | `UUID` |
| 310 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 311 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 최종 수정 사용자 | `UUID` |
| 312 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 조건 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 업무 규칙

전체 활동 승인 조건은 선택된 실제 수행 활동만 평가한다. RTM은 선행·후행 활동 또는 독립 승인 조건으로 등록하지 않는다.

STATUS 조건에서는 predecessor_activity_id와 required_status가 필수이며 required_status는 CREATED/COMPLETED/APPROVED이다. 다른 조건에서는 필요하지 않은 선행 활동·상태 값은 NULL로 둔다. 활성 선후행 관계에 순환을 허용하지 않는다.

이 테이블은 신규 프로젝트에 적용할 전역 조건 템플릿이다. 변경 시 새로운 조건 묶음 버전을 발행하고 전체 조건과 판정 의미를 변경 불가능한 버전별 설정으로 보존한다. 프로젝트 생성 시 선택한 버전의 전체 내용을 validation_project.dependency_snapshot에 복사하며, 이후 조건의 수정·비활성화·삭제는 기존 프로젝트의 조건을 변경하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-project_closure_request"></a>
### 24. 프로젝트 종료 요청 (`project_closure_request`)

| 항목 | 정의 |
|---|---|
| 설명 | 정상종료·강제종료 사유와 요청별 검토·승인 진행 |
| Primary Key | `closure_request_id` |
| 주요 참조(FK) | `project_id, requested_by, workflow_instance_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 313 | 종료 요청 ID | `closure_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 314 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 315 | 요청 순번 | `request_version` | `integer` | N | N | - | Y | - | N | N | N | Y | 같은 프로젝트의 종료 재요청 구분 | `1` |
| 316 | 종료 유형 | `closure_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | NORMAL=정상종료, FORCED=강제종료 | `NORMAL` |
| 317 | 요청 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=작성중, REVIEW=검토중, APPROVAL=승인중, APPROVED=완료, REJECTED=반려, CANCELLED=취소 | `REVIEW` |
| 318 | 종료 사유 | `reason` | `text` | N | N | - | N | - | N | N | N | Y | 강제종료일 때 필수 | - |
| 319 | 요청 시 진행률 | `progress_snapshot` | `numeric(5,2)` | N | N | - | Y | - | N | N | N | Y | 종료 확인 화면에 표시한 요청 당시 진행률. RTM 제외 | `100.00` |
| 320 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 321 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 요청 시각 값 | `2026-09-01T00:00:00Z` |
| 322 | 종료 결재 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 323 | 종료 처리 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 종료 처리 시각 값 | `2026-09-01T00:00:00Z` |
| 324 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 325 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 326 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 327 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_closure_request_1` | UNIQUE | `project_id, request_version` | UNIQUE (project_id, request_version) |
| `ck_project_closure_request_2` | CHECK | `request_version` | CHECK (request_version >= 1) |
| `ck_project_closure_request_3` | CHECK | `progress_snapshot` | CHECK (progress_snapshot BETWEEN 0 AND 100) |
| `rule_project_closure_request_4` | 업무 검증 | `closure_type` | closure_type='FORCED'이면 reason은 비어 있을 수 없다. |
| `rule_project_closure_request_5` | 업무 검증 | - | 동일 프로젝트의 DRAFT/REVIEW/APPROVAL 상태 종료 요청은 최대 한 건이다. |
| `uq_project_closure_active` | UNIQUE | `project_id` | UNIQUE (project_id) WHERE approval_status IN ('DRAFT','REVIEW','APPROVAL') |

#### 업무 규칙

정상종료는 선택된 수행 활동과 필요한 산출물의 완료·승인 상태를 확인한다. RTM 독립 승인과 RTM 산출물은 종료 조건에서 제외한다.

서명자는 전자서명 및 결재 처리 이력으로 연결한다. workflow_instance의 대상은 project_closure_request/closure_request_id/CLOSE-{request_version}이고 프로젝트가 일치해야 한다. 최종 승인 후 validation_project.status와 종료 요약을 함께 반영하며 검토중/승인중은 프로젝트 종료 상태로 처리하지 않는다.

[↑ 맨 위로](#top)

---

## VP

<a id="table-vp_plan"></a>
### 25. 검증 계획 (`vp_plan`)

| 항목 | 정의 |
|---|---|
| 설명 | VP의 목차·본문 편집 및 업무 승인 개정 관리. |
| Primary Key | `vp_id` |
| 주요 참조(FK) | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 328 | VP 개정 ID | `vp_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 329 | 개정 간 논리 ID | `vp_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 330 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 331 | 계획 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 검증 계획의 표시 제목 | `시스템 검증 계획` |
| 332 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 333 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 334 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 335 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 336 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 337 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 338 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 339 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 340 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 341 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 342 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 343 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 344 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 345 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vp_plan_1` | UNIQUE | `vp_key, revision_number` | UNIQUE (vp_key, revision_number) |
| `uq_vp_plan_1_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vp_plan_2` | UNIQUE | `vp_key, is_current_version` | UNIQUE (vp_key) WHERE is_current_version = TRUE |
| `ck_vp_plan_3` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_vp_plan_4` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vp_plan_4_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |
| `rule_vp_plan_5` | 업무 검증 | `workflow_instance_id` | 프로젝트당 VP 논리키는 하나이며 개정은 여러 건이다. APPROVED에는 workflow_instance_id 필수. |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

목차 추가·삭제·정렬·포함 여부·본문을 vp_section에 저장한다. 생성 산출물의 편집 내용과 문서 승인 상태는 deliverable_revision에서 관리하며 원본 VP와 구분한다.

같은 vp_key의 모든 개정은 동일 프로젝트에 속한다. 개정 생성과 최신 개정 전환은 프로젝트 및 논리 항목을 잠그고 같은 트랜잭션에서 처리한다.

[↑ 맨 위로](#top)

---

<a id="table-vp_section"></a>
### 26. 검증 계획 목차 (`vp_section`)

| 항목 | 정의 |
|---|---|
| 설명 | VP 개정별 목차 제목·내용·표시 순서·포함 여부. |
| Primary Key | `vp_section_id` |
| 주요 참조(FK) | `vp_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 346 | 목차 ID | `vp_section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 347 | VP 개정 ID | `vp_id` | `uuid` | N | Y | `vp_plan.vp_id` | Y | - | N | Y | N | Y | vp_plan.vp_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 348 | 목차 논리 키 | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 개정 간 유지하는 목차 식별 키 | `purpose` |
| 349 | 목차 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 목차 제목 | `목적` |
| 350 | 목차 내용 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | 목차 내용 | `본 계획의 검증 목적` |
| 351 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 352 | 산출물 포함 여부 | `is_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 산출물 포함 여부 | `TRUE` |
| 353 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 354 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 355 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 356 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vp_section_1` | UNIQUE | `vp_id, section_key` | UNIQUE (vp_id, section_key) |
| `uq_vp_section_1_2` | UNIQUE | `vp_id, sort_order` | UNIQUE (vp_id, sort_order) |
| `uq_vp_section_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |

#### 업무 규칙

승인된 VP의 하위 목차는 직접 수정하지 않으며 새 VP 개정에 복사한다. AI 초안 생성 후 사용자가 저장한 내용도 content에 기록한다.

[↑ 맨 위로](#top)

---

## QIA

<a id="table-qia_assessment"></a>
### 27. 품질 영향 평가 헤더 (`qia_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 QIA 모듈 목록과 Part 11 공통 질문 응답을 관리하는 평가 헤더. |
| Primary Key | `qia_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 357 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 평가 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 358 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 359 | Part 11 Q1 전자기록의 종이기록 대체 여부 | `p11_q1` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 전자 기록이 종이 기록을 대체합니까? 응답 Yes / No | `No` |
| 360 | Part 11 Q2 전자서명 사용 여부 | `p11_q2` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 전자 서명을 사용합니까? 응답 Yes / No | `No` |
| 361 | Part 11 Q3 기록 이력의 규제 증빙 여부 | `p11_q3` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 기록 생성·변경 이력이 규제 증빙입니까? 응답 Yes / No | `No` |
| 362 | Part 11 Q4 접근 통제·사용자 식별 필요 여부 | `p11_q4` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 접근 통제와 사용자 식별이 필요합니까? 응답 Yes / No | `No` |
| 363 | Part 11 Q5 장기 보관·검색 필요 여부 | `p11_q5` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 기록의 장기 보관과 검색이 필요합니까? 응답 Yes / No | `No` |
| 364 | Part 11 Q6 시스템 간 전자기록 전송 여부 | `p11_q6` | `varchar(3)` | N | N | - | Y | `'No'` | N | N | N | Y | 시스템 간 전자 기록 전송이 있습니까? 응답 Yes / No | `No` |
| 365 | Part11 평가 결론 | `part11_result` | `text` | N | N | - | N | - | N | N | N | N | 6개 응답에서 자동 계산한 표시값. 하나라도 Yes이면 적용, 모두 No이면 비적용. 직접 입력하지 않음 | `Part 11 비적용` |
| 366 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 367 | 질문 세트 버전 | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | 6개 질문 문구·순서를 고정하는 버전 | `UI-P11-1` |
| 368 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | 응답 중 하나 이상이 Yes이면 적용으로 판정하는 규칙의 버전 | `UI-P11-1` |
| 369 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 370 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 371 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 372 | 적용 질문 원문 | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. 선택한 버전의 모든 질문·문구·순서·응답 선택값을 저장한다. | - |
| 373 | 적용 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. 논리식·응답 해석·판정값 매핑을 포함한 전체 정의 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_qia_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |
| `rule_qia_assessment_2` | 업무 검증 | `p11_q1` | p11_q1~p11_q6는 'Yes' 또는 'No'. part11_result는 응답과 같은 트랜잭션에서 재계산하는 캐시이며 독립 수정 금지. |

#### 업무 규칙

Part 11 질문은 모듈별이 아닌 QIA 공통 응답이다. QIA 문서 생성·승인 시 질문 버전·응답·규칙 버전·결과를 deliverable_revision의 원본/표 스냅샷에 고정한다. 현재 응답 변경으로 과거 승인 문서가 바뀌지 않는다.

문서의 승인 상태와 버전은 deliverable_document 및 deliverable_revision에서 관리한다. 모듈별 GxP 판정은 하위 프로세스에서 조회한다.

평가 작성 시 버전별 질문·규칙 정의를 복사하고 version 값이 question_set_version 및 rule_version과 일치하는지 서버에서 검증한다. 동일 버전 이름에 다른 정의를 등록하지 않는다. 새 전역 버전을 기존 평가에 자동 적용하지 않는다. 계산에는 저장한 정의를 사용하며 단순한 함수명이나 버전 문자열만으로 원문을 대체하지 않는다.

Part 11 공통 응답 또는 적용 정의를 변경해도 이미 상신·승인한 문서의 evaluation_definition_snapshot은 변경하지 않는다. QIA 문서 상신 시 공통 평가와 포함된 모든 프로세스의 전체 질문·규칙·응답·판정, 프로세스가 없는 모듈의 판정 정책을 evaluation_definition_snapshot에 함께 고정한다.

[↑ 맨 위로](#top)

---

<a id="table-qia_module_item"></a>
### 28. QIA 모듈 상세 평가 (`qia_module_item`)

| 항목 | 정의 |
|---|---|
| 설명 | QIA 모듈 이름·설명과 모듈별 승인·폐기·개정 관리. |
| Primary Key | `qia_module_item_id` |
| 주요 참조(FK) | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 374 | QIA 모듈 항목 ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 모듈 평가 항목 고유 식별자 | `UUID` |
| 375 | 개정 간 논리 ID | `module_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 376 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | 상위 QIA 평가 식별자 | `UUID` |
| 377 | 모듈 코드 | `module_code` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | 선택 내부 모듈 코드. 화면 승인번호 item_number와 구분 | `MOD-001` |
| 378 | 모듈명 | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 상위 모듈명 | `품질관리` |
| 379 | 모듈 설명 | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | 모듈의 범위 및 목적 설명 | `품질관리 관련 종합 평가` |
| 380 | 평가 결과 | `result_type` | `varchar(20)` | N | N | - | N | - | N | N | N | N | 프로세스 판정의 집계 캐시. 한 프로세스라도 GXP이면 GXP, 아니면 NON_GXP. 직접 편집하지 않음 | `GXP` |
| 381 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-26T10:00:00Z` |
| 382 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-26T10:00:00Z` |
| 383 | 모듈 표시 순서 | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 모듈 표시 순서 | `1` |
| 384 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 385 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 386 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 387 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 388 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 389 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 390 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `QIA-001` |
| 391 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 392 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 393 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 394 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 395 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 396 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_qia_module_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_qia_module_item_2` | UNIQUE | `module_key, revision_number` | UNIQUE (module_key, revision_number) |
| `uq_qia_module_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_qia_module_item_3` | UNIQUE | `module_key, is_current_version` | UNIQUE (module_key) WHERE is_current_version = TRUE |
| `ck_qia_module_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_qia_module_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_qia_module_item_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |
| `ck_qia_module_item_6` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `ck_qia_module_item_6_rule` | 업무 검증 | `sort_order` | 모듈 먼저 등록 후 프로세스를 추가할 수 있으므로 하위 프로세스 0건 초안을 허용한다 |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

프로세스가 없는 모듈의 판정은 NON_GXP이며, 프로세스 존재 여부는 승인 제한 조건이 아니다. NON_GXP 표시만으로 실제 프로세스 평가 완료를 뜻하지 않는다.

모듈과 프로세스는 1:N 관계이며 프로세스명과 10개 질문 응답은 qia_process에 저장한다. 개정 시 프로세스도 새 모듈 개정 아래 복제하며 승인 모듈의 프로세스 원문·응답은 불변이다.

같은 module_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-qia_process"></a>
### 29. QIA 프로세스 평가 (`qia_process`)

| 항목 | 정의 |
|---|---|
| 설명 | 모듈 개정 아래 프로세스명·설명과 GxP 10개 질문 응답. |
| Primary Key | `qia_process_id` |
| 주요 참조(FK) | `qia_module_item_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 397 | 프로세스 ID | `qia_process_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 398 | 모듈 개정 ID | `qia_module_item_id` | `uuid` | N | Y | `qia_module_item.qia_module_item_id` | Y | - | N | Y | N | Y | qia_module_item.qia_module_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 399 | 프로세스 논리 ID | `process_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | N | N | Y | 모듈 개정 간 유지하는 프로세스 식별자 | `00000000-0000-0000-0000-000000000001` |
| 400 | 내부 프로세스 코드 | `process_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 내부 프로세스 코드 | - |
| 401 | 프로세스명 | `process_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 프로세스명 | `시험 결과 입력` |
| 402 | 프로세스 설명 | `process_description` | `text` | N | N | - | N | - | N | N | N | Y | 프로세스 설명 | - |
| 403 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 표시 순서 | `1` |
| 404 | 질문 세트 버전 | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | 질문 세트 버전 | `UI-GXP-1` |
| 405 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | 판정 규칙 버전 | `UI-GXP-1` |
| 406 | GxP Q1 | `q1_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 제품 품질, 환자 안전 또는 데이터 무결성에 직접 영향을 주는가? 응답 O / X / ▲ | `X` |
| 407 | GxP Q2 | `q2_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | GxP 의사결정에 사용되는 데이터를 생성하거나 처리하는가? 응답 O / X / ▲ | `X` |
| 408 | GxP Q3 | `q3_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 배치 출하 또는 품질 승인 절차를 지원하는가? 응답 O / X / ▲ | `X` |
| 409 | GxP Q4 | `q4_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 규제기관 제출 또는 검사 증빙에 사용되는가? 응답 O / X / ▲ | `X` |
| 410 | GxP Q5 | `q5_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 전자 기록의 생성·수정·보관을 수행하는가? 응답 O / X / ▲ | `X` |
| 411 | GxP Q6 | `q6_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 장비 또는 공정의 중요 파라미터를 제어하는가? 응답 O / X / ▲ | `X` |
| 412 | GxP Q7 | `q7_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 일탈, CAPA, 변경 관리 프로세스를 지원하는가? 응답 O / X / ▲ | `X` |
| 413 | GxP Q8 | `q8_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 사용자 권한이 품질 관련 기능 접근을 통제하는가? 응답 O / X / ▲ | `X` |
| 414 | GxP Q9 | `q9_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 외부 GxP 시스템과 중요 데이터를 교환하는가? 응답 O / X / ▲ | `X` |
| 415 | GxP Q10 | `q10_val` | `varchar(5)` | N | N | - | Y | `'X'` | N | N | N | Y | 백업·복구 실패가 GxP 기록에 영향을 주는가? 응답 O / X / ▲ | `X` |
| 416 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 417 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 418 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 419 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 420 | 적용 질문 원문 | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. 선택한 버전의 모든 질문·문구·순서·응답 선택값을 저장한다. | - |
| 421 | 적용 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. 논리식·응답 해석·판정값 매핑을 포함한 전체 정의 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_qia_process_1` | UNIQUE | `qia_module_item_id, process_key` | UNIQUE (qia_module_item_id, process_key) |
| `uq_qia_process_1_2` | UNIQUE | `qia_module_item_id, sort_order` | UNIQUE (qia_module_item_id, sort_order) |
| `uq_qia_process_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_process_2` | 업무 검증 | `q1_val` | q1_val~q10_val는 O/X/▲만 허용한다. |

#### 업무 규칙

GxP 판정 규칙: Q1=O이면서 Q2~Q10 중 O가 하나 이상이면 GXP, 그 외 NON_GXP. ▲는 O 조건에 포함하지 않는다. 판정은 응답에서 조회 계산한다.

승인 모듈에 종속된 질문 버전·응답은 직접 덮어쓰지 않는다.

평가 작성 시 버전별 질문·규칙 정의를 복사하고 version 값이 question_set_version 및 rule_version과 일치하는지 서버에서 검증한다. 동일 버전 이름에 다른 정의를 등록하지 않는다. 새 전역 버전을 기존 평가에 자동 적용하지 않는다. 계산에는 저장한 정의를 사용하며 단순한 함수명이나 버전 문자열만으로 원문을 대체하지 않는다.

승인 모듈의 질문·규칙·응답을 변경하려면 모듈을 새 개정하고 프로세스 및 적용 정의를 복사한다. 과거 모듈에 속한 원문은 그대로 보존한다.

[↑ 맨 위로](#top)

---

## VA

<a id="table-vendor_audit"></a>
### 30. 공급업체 감사 평가 (`vendor_audit`)

| 항목 | 정의 |
|---|---|
| 설명 | 공급업체명·감사일·감사자·첨부파일과 평가별 승인·개정 관리. |
| Primary Key | `audit_id` |
| 주요 참조(FK) | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 422 | 감사 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 공급업체 감사 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 423 | 개정 간 논리 ID | `audit_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 424 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 425 | 문서 번호 | `document_number` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `VA-001` |
| 426 | 공급업체명 | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사 대상 공급업체명 | `Sample Vendor` |
| 427 | 감사 일자 | `audit_date` | `date` | N | N | - | Y | `CURRENT_DATE` | N | N | N | Y | 저장 시 화면이 현재 날짜로 자동 기록하는 감사일 | `2026-09-16` |
| 428 | 감사자 | `auditor_name` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 저장 당시 로그인 감사자 표시명. 사용자 이름이 바뀌어도 이력 보존 | `홍길동 · reviewer@example.test` |
| 429 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 430 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 431 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 432 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 433 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 434 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 435 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 436 | 감사자 계정 ID | `auditor_user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 저장 당시 로그인 계정 | `00000000-0000-0000-0000-000000000001` |
| 437 | 평가 첨부파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 선택 업로드 첨부파일. 파일명·크기·경로는 파일 자산에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 438 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 439 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 440 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 441 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 442 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 443 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 444 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_vendor_audit_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 document_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_vendor_audit_2` | UNIQUE | `audit_key, revision_number` | UNIQUE (audit_key, revision_number) |
| `uq_vendor_audit_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vendor_audit_3` | UNIQUE | `audit_key, is_current_version` | UNIQUE (audit_key) WHERE is_current_version = TRUE |
| `ck_vendor_audit_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_vendor_audit_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vendor_audit_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

시스템명은 프로젝트가 연결한 시스템에서 조회한다. 첨부파일은 선택 항목이며 NULL을 허용한다.

같은 audit_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 document_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## URS

<a id="table-requirement"></a>
### 31. 사용자 요구사항 명세 (`requirement`)

| 항목 | 정의 |
|---|---|
| 설명 | 화면 요구사항의 카테고리·항목명·내용·수용 기준·규정 근거·등록 출처 및 승인 개정 관리. |
| Primary Key | `requirement_id` |
| 주요 참조(FK) | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 445 | 요구사항 ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 요구사항 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 446 | 개정 간 논리 ID | `requirement_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 447 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 448 | 항목 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `URS-001` |
| 449 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 시스템 관리, 감사추적, 전자서명 등 | `전자서명` |
| 450 | 항목 / 기능 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 요구사항 항목명 및 주요 기능 | `전자서명 서명자·일시·의미 기록` |
| 451 | 요구사항 상세 | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | 요구사항 상세 명세 내용 | `전자서명 시 서명자 ID, 서명 일시...` |
| 452 | 수용 기준 | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | 화면 수용 기준 입력 | `정의한 권한별 접근이 제한된다` |
| 453 | 근거 규정 | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | 수기 규정 근거 또는 마스터 미연결 원문. 정규 조항 연결은 requirement_regulation에서 관리 | `CSV 규정 근거 검토 메모` |
| 454 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 455 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 456 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 457 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 458 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 459 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 요구사항 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 460 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 461 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 462 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 463 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 464 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 465 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 466 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 467 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 468 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 469 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 470 | 기타 출처 식별값 | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지 또는 AI 초안의 식별값. 외부/다형 출처이므로 DB FK 아님 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_requirement_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_requirement_2` | UNIQUE | `requirement_key, revision_number` | UNIQUE (requirement_key, revision_number) |
| `uq_requirement_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_requirement_3` | UNIQUE | `requirement_key, is_current_version` | UNIQUE (requirement_key) WHERE is_current_version = TRUE |
| `ck_requirement_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_requirement_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_requirement_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |
| `ck_requirement_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_requirement_6_rule` | 업무 검증 | `source_type, source_library_id` | LIBRARY이면 source_library_id 필수 |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

라이브러리/시스템 패키지/AI에서 가져온 내용은 이 항목의 초안으로 복사하여 편집한다. 출처 변경이 승인된 업무 항목에 자동 반영되지 않는다.

requirement_id는 특정 개정행 PK이다. DQ/FRA/시험/규정은 해당 개정행에 연결한다. requirement_key별 최신 초안과 최신 승인 개정을 구분한다.

규정 조항은 requirement_regulation으로 연결하고 개정 복제 시 인용도 복사한다. RTM의 연결 상태는 업무 관계와 판정 결과에서 조회한다.

같은 requirement_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## FDS

<a id="table-fds_spec"></a>
### 32. 기능 설계 명세서 (`fds_spec`)

| 항목 | 정의 |
|---|---|
| 설명 | F&DS 그룹의 FDS 업로드 문서와 파일 개정·승인 이력. 프로젝트당 여러 문서를 허용. |
| Primary Key | `fds_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 471 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FDS 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 472 | 개정 간 논리 ID | `fds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 473 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 474 | FDS 번호 | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `FDS-001` |
| 475 | FDS 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 문서의 표시 제목으로 업로드 파일명을 사용 | `FDS_ver1.pdf` |
| 476 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 477 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 478 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 479 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 480 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 481 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 482 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 483 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 484 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 485 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 486 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 487 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 488 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 489 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 490 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 491 | 업로드 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | 해당 설계 개정에 업로드한 파일. 교체하면 새 개정행과 새 파일 ID 생성 | `00000000-0000-0000-0000-000000000001` |
| 492 | 업로드 시각 | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이 파일 개정을 업로드한 시각 | `2026-09-01T00:00:00Z` |
| 493 | 업로더 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_fds_spec_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 fds_no와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_fds_spec_2` | UNIQUE | `fds_key, revision_number` | UNIQUE (fds_key, revision_number) |
| `uq_fds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fds_spec_3` | UNIQUE | `fds_key, is_current_version` | UNIQUE (fds_key) WHERE is_current_version = TRUE |
| `ck_fds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_fds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fds_spec_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

F&DS는 하나의 수행 활동이며 문서 종류의 표시 순서는 FDS 다음 DDS이다. DDS에 FDS 선행 승인 또는 단일 부모 FDS를 필수로 강제하지 않는다.

파일 교체 이력은 같은 논리키의 이전 개정행에서 조회한다. DQ는 이 테이블 PK를 정확한 승인 파일 개정으로 참조한다.

같은 fds_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 fds_no는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## DDS

<a id="table-dds_spec"></a>
### 33. 상세 설계 명세서 (`dds_spec`)

| 항목 | 정의 |
|---|---|
| 설명 | F&DS 그룹의 DDS 업로드 문서와 파일 개정·승인 이력. 프로젝트당 여러 문서를 허용. |
| Primary Key | `dds_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 494 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DDS 문서 고유 식별자 | `UUID` |
| 495 | 개정 간 논리 ID | `dds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 496 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | DDS가 속한 Validation 프로젝트 | `UUID` |
| 497 | DDS 번호 | `dds_no` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `DDS-001` |
| 498 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 문서의 표시 제목으로 업로드 파일명을 사용 | `DDS_ver1.pdf` |
| 499 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 500 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 501 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 502 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 503 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 504 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 505 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 작성 사용자 | `UUID` |
| 506 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 507 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 최종 수정 사용자 | `UUID` |
| 508 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 509 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 510 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 511 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 512 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 513 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 514 | 업로드 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | 해당 설계 개정에 업로드한 파일. 교체하면 새 개정행과 새 파일 ID 생성 | `00000000-0000-0000-0000-000000000001` |
| 515 | 업로드 시각 | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이 파일 개정을 업로드한 시각 | `2026-09-01T00:00:00Z` |
| 516 | 업로더 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_dds_spec_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 dds_no와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_dds_spec_2` | UNIQUE | `dds_key, revision_number` | UNIQUE (dds_key, revision_number) |
| `uq_dds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dds_spec_3` | UNIQUE | `dds_key, is_current_version` | UNIQUE (dds_key) WHERE is_current_version = TRUE |
| `ck_dds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_dds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dds_spec_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

F&DS는 하나의 수행 활동이며 문서 종류의 표시 순서는 FDS 다음 DDS이다. DDS에 FDS 선행 승인 또는 단일 부모 FDS를 필수로 강제하지 않는다.

파일 교체 이력은 같은 논리키의 이전 개정행에서 조회한다. DQ는 이 테이블 PK를 정확한 승인 파일 개정으로 참조한다.

같은 dds_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 dds_no는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## DQ

<a id="table-dq_assessment"></a>
### 34. 설계 적격성 평가 (`dq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | DQ 평가 항목을 묶는 프로젝트별 헤더. |
| Primary Key | `dq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 517 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 518 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 519 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 적격성 평가 문서 제목 | `설계 적격성 평가 (URS → FDS/DDS 매핑)` |
| 520 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 521 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 522 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 523 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 524 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_dq_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### 업무 규칙

항목별 업무 승인/개정은 자식 테이블에서 관리하고 생성 문서의 번호·버전·승인 상태는 deliverable_document/deliverable_revision에서 관리한다. 헤더의 승인 상태를 하위 항목 전체에 일괄 복사하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-dq_item"></a>
### 35. DQ 상세 평가 항목 (`dq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 승인 URS와 FDS/DDS 파일 개정을 대조한 코멘트·판정·Fail 사유·수행 및 항목 승인 개정 관리. |
| Primary Key | `dq_item_id` |
| 주요 참조(FK) | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 525 | DQ 항목 ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 526 | 개정 간 논리 ID | `dq_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 527 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | Y | N | Y | 상위 DQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 528 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | 매핑 대상 URS 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 529 | URS 번호 | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | requirement_id가 가리키는 승인 개정의 번호 스냅샷. 직접 입력하지 않음 | `URS-001` |
| 530 | URS 요구사항 | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | requirement_id가 가리키는 승인 개정의 요구사항 내용 스냅샷. 직접 입력하지 않음 | `사용자 로그인 및 전자서명 기능` |
| 531 | 판정 | `result_status` | `varchar(10)` | N | N | - | N | - | N | N | N | Y | PASS / FAIL / N/A. 미판정은 NULL | `PASS` |
| 532 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 검토 관련 특이사항 및 비고 | `FDS 및 DDS 설계 반영 완료 확인` |
| 533 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 534 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 535 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 536 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 537 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 538 | FDS 승인 개정 ID | `fds_revision_id` | `uuid` | N | Y | `fds_spec.fds_id` | N | - | N | Y | N | Y | fds_spec.fds_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 539 | FDS 검토 코멘트 | `fds_comment` | `text` | N | N | - | N | - | N | N | N | Y | FDS 검토 코멘트 | - |
| 540 | DDS 승인 개정 ID | `dds_revision_id` | `uuid` | N | Y | `dds_spec.dds_id` | N | - | N | Y | N | Y | dds_spec.dds_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 541 | DDS 검토 코멘트 | `dds_comment` | `text` | N | N | - | N | - | N | N | N | Y | DDS 검토 코멘트 | - |
| 542 | Fail 사유 | `fail_reason` | `text` | N | N | - | N | - | N | N | N | Y | Fail 사유 | - |
| 543 | 판정 수행자 ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 544 | 판정 수행 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 판정 수행 시각 | `2026-09-01T00:00:00Z` |
| 545 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 546 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 547 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 548 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 549 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 550 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 551 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `DQ-001` |
| 552 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 553 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 554 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 555 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_dq_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_dq_item_2` | UNIQUE | `dq_item_key, revision_number` | UNIQUE (dq_item_key, revision_number) |
| `uq_dq_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dq_item_3` | UNIQUE | `dq_item_key, is_current_version` | UNIQUE (dq_item_key) WHERE is_current_version = TRUE |
| `ck_dq_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_dq_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dq_item_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |
| `ck_dq_item_6` | CHECK | `result_status` | CHECK (result_status IS NULL OR result_status IN ('PASS','FAIL','N/A')) |
| `ck_dq_item_6_rule` | 업무 검증 | `result_status, fail_reason` | FAIL이면 공백이 아닌 fail_reason 필수 |
| `rule_dq_item_7` | 업무 검증 | `result_status, executed_by, executed_at` | 승인 요청 시 result_status·executed_by·executed_at 필수이며 FDS/DDS 중 하나 이상 같은 프로젝트의 유효 승인 개정이어야 한다. requirement_id도 같은 프로젝트의 승인 개정이어야 한다. |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

fds_revision_id와 dds_revision_id는 승인된 설계 문서의 정확한 개정행을 참조한다. 파일명·문서번호·버전은 연결한 fds_spec/dds_spec에서 조회한다.

수행자는 executed_by로 저장하며 검토·승인자는 workflow_instance와 approval_action에서 조회한다. N/A 판정도 미판정과 구분하고 설계 연결을 확인한다. DQ FAIL은 시험 일탈을 자동 생성하지 않는다.

같은 dq_item_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## FRA

<a id="table-fra_assessment"></a>
### 36. 기능 위험평가 (`fra_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | FRA 평가 항목을 묶는 프로젝트별 헤더. |
| Primary Key | `fra_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 556 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 557 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 558 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | FMEA 기반 기능 위험평가 문서 제목 | `FMEA 기반 기능 위험평가` |
| 559 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 560 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 561 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 562 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 563 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_fra_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### 업무 규칙

항목별 업무 승인/개정은 자식 테이블에서 관리하고 생성 문서의 번호·버전·승인 상태는 deliverable_document/deliverable_revision에서 관리한다. 헤더의 승인 상태를 하위 항목 전체에 일괄 복사하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-fra_item"></a>
### 37. FRA 위험 상세 항목 (`fra_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 승인 URS와 연결한 기능·위험 시나리오·위험 점수·등록 출처 및 항목 승인 개정 관리. |
| Primary Key | `fra_item_id` |
| 주요 참조(FK) | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 564 | 위험 항목 ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA 위험 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 565 | 개정 간 논리 ID | `fra_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 566 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | Y | N | Y | 상위 FRA 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 567 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | Y | N | Y | 동일 프로젝트의 승인된 URS 개정. 초안 NULL 허용, 승인 요청 전 필수 | `00000000-0000-0000-0000-000000000001` |
| 568 | URS 참조번호 | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 연결한 요구사항 개정의 승인번호 표시 스냅샷. requirement_id와 일치하며 독립 입력하지 않음 | `URS-001` |
| 569 | 기능명 | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 위험 평가 대상 기능명 | `전자서명` |
| 570 | 위험 시나리오 | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | FMEA 위험 발생 시나리오 설명 | `전자서명 시 비밀번호 검증 미수행` |
| 571 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) | `DRAFT` |
| 572 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 573 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 574 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 575 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 576 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 577 | 심각도 SEV | `severity` | `smallint` | N | N | - | Y | `1` | N | N | N | Y | 화면 입력 1~5 | `5` |
| 578 | 발생가능성 OCC | `occurrence` | `smallint` | N | N | - | Y | `1` | N | N | N | Y | 화면 입력 1~5 | `2` |
| 579 | 검출도 DET | `detectability` | `varchar(1)` | N | N | - | Y | `'M'` | N | N | N | Y | H / M / L | `M` |
| 580 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-FRA-1'` | N | N | N | Y | 판정 규칙 버전 | `UI-FRA-1` |
| 581 | 승인 시 계산 결과 | `risk_result_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | 승인 당시 RP·RC·RPG·NT·Action Plan 자동 결과. 직접 입력하지 않음 | `{"RP":10,"RC":1,"RPG":"H","NT":"N","actionPlan":"Test 수행"}` |
| 582 | SOP 근거 조항 ID | `sop_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | N | - | N | Y | N | Y | RTM 화면의 SOP 연결을 원본 위험 항목에 저장 | `00000000-0000-0000-0000-000000000001` |
| 583 | 수기 SOP 참조 | `sop_reference_note` | `text` | N | N | - | N | - | N | N | N | Y | 조항 마스터에 아직 없는 SOP 참조 원문 | - |
| 584 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 585 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 586 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유 | `요구사항 변경` |
| 587 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 588 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 589 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `FRA-001` |
| 590 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 591 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 592 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 593 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 594 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 595 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 596 | 기타 출처 식별값 | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지 또는 AI 초안의 식별값. 외부/다형 출처이므로 DB FK 아님 | - |
| 597 | 적용 위험 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_ranges,formulas,thresholds,matrix,action_mapping}. SEV/OCC/DET 허용값, RP·RC 계산식, RPG·NT 판정표와 Action Plan 매핑 전체 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_fra_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_fra_item_2` | UNIQUE | `fra_item_key, revision_number` | UNIQUE (fra_item_key, revision_number) |
| `uq_fra_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fra_item_3` | UNIQUE | `fra_item_key, is_current_version` | UNIQUE (fra_item_key) WHERE is_current_version = TRUE |
| `ck_fra_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_fra_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fra_item_5_rule` | 업무 검증 | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | DISPOSED에는 disposed_at·disposal_reason·disposal_workflow_id 필수 |
| `ck_fra_item_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_fra_item_6_rule` | 업무 검증 | `source_type, source_library_id` | LIBRARY이면 source_library_id 필수 |
| `ck_fra_item_7` | CHECK | `severity` | CHECK (severity BETWEEN 1 AND 5) |
| `ck_fra_item_7_2` | CHECK | `occurrence` | CHECK (occurrence BETWEEN 1 AND 5) |
| `ck_fra_item_7_3` | CHECK | `detectability` | CHECK (detectability IN ('H','M','L')) |
| `rule_fra_item_8` | 업무 검증 | `requirement_id, risk_result_snapshot` | 승인 요청 전 requirement_id 필수이고 해당 개정이 같은 프로젝트의 유효 승인본인지 검증한다. APPROVED에는 workflow_instance_id와 risk_result_snapshot 필수. |
| `rule_fra_item_9` | 업무 검증 | - | sop_clause_id는 INTERNAL_SOP 유형의 조항이어야 한다. |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

라이브러리/시스템 패키지/AI에서 가져온 내용은 이 항목의 초안으로 복사하여 편집한다. 출처 변경이 승인된 업무 항목에 자동 반영되지 않는다.

위험 평가 계산 규칙: RP=SEV×OCC; RC는 RP<5→3, RP<10→2, 나머지→1. RPG는 RC1의 DET H/M/L→M/H/H, RC2→L/M/H, RC3→L/L/M. NT는 RP>24→Y, 그 외 N. Action Plan은 RPG L→No Action, M→SOP 수정/삭제, H→Test 수행.

초안 결과는 조회 계산하고 승인 당시 결과만 스냅샷으로 보존한다. 위험 점수와 자동 조치계획은 판정 규칙으로 산출하며 독립 입력값으로 저장하지 않는다. 시험은 실제 시험/FRA 연결 관계로 추적한다.

작성 시 rule_version의 실제 정의를 rule_snapshot에 복사하고 해당 정의로 결과를 계산한다. 승인 후 규칙·입력·결과를 변경하지 않으며 변경 시 새 항목 개정을 생성한다. 같은 버전 이름의 정의는 변경하지 않는다. FRA 문서 상신 시 근거 항목별 정의와 입력·risk_result_snapshot을 evaluation_definition_snapshot에 고정한다.

같은 fra_item_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## IQ

<a id="table-iq_assessment"></a>
### 38. 설치 적격성 평가 (`iq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 IQ 시험 묶음 및 문서 개정 정보. 프로토콜과 수행 결과의 승인 상태를 구분한다. |
| Primary Key | `iq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 598 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 599 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 600 | IQ 번호 | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `IQ-VP-SYS-010-20260529` |
| 601 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설치 적격성 평가 문서 제목 | `Installation Qualification` |
| 602 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 603 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 604 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | IQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 605 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 IQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 606 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 607 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 608 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 609 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 610 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 611 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | IQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 612 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 613 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"iq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_assessment_1` | UNIQUE | `project_id, iq_no, revision_number` | UNIQUE(project_id, iq_no, revision_number) |
| `uq_iq_assessment_1_rule` | 업무 검증 | `project_id, iq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_iq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_iq_assessment_current` | UNIQUE | `project_id, iq_no` | UNIQUE (project_id, iq_no) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 iq_item의 실제 개정이며 같은 프로젝트·IQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 iq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다.

[↑ 맨 위로](#top)

---

<a id="table-iq_item"></a>
### 39. IQ 상세 테스트 항목 (`iq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `iq_item_id` |
| 주요 참조(FK) | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 614 | IQ 항목 ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 615 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | N | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 616 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 617 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `IQ-NEW-01` |
| 618 | 테스트 케이스 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. 별도 분류값이 아님. | `하드웨어 설치` |
| 619 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `설치될 서버의 하드웨어 사양이 URS를 충족하는지 확인` |
| 620 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `하드웨어 사양이 URS에 명시된 요구사항과 일치해야 함` |
| 621 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 이 시험 개정의 독립적인 승인 상태이며 상위 문서 상태로 덮어쓰지 않음. | `DRAFT` |
| 622 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 623 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 624 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 625 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 626 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 627 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 628 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 629 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 630 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 631 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 632 | 등록 출처 | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), PACKAGE(시스템 패키지), AI(AI 초안). | `MANUAL` |
| 633 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 634 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `IQ-CORE` |
| 635 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 636 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `IQ-PKG-001` |
| 637 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 638 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 639 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 640 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 권한자가 사유와 대상을 확인하여 수행한 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 641 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 642 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 643 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 화면 폐기 완료 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_iq_item_1_rule` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 IQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_iq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 승인 완료된 프로토콜과 절차는 직접 수정하지 않고 새 개정 행을 생성한다. |
| `rule_iq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_iq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_iq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → IQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_iq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_iq_item_7` | 업무 검증 | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | is_disposed = TRUE이면 disposal_reason, disposal_signature_id, disposed_by, disposed_at 필수. 폐기 서명의 실제 대상 테이블·PK는 이 시험 개정이고 서명자·시각은 disposed_by / disposed_at과 일치해야 한다. |
| `rule_iq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_iq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 iq_step, 실제 결과·서명·재수행 이력은 iq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-iq_step"></a>
### 40. IQ 시험 절차 (`iq_step`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 프로토콜 개정에 포함된 순서 있는 세부 절차 |
| Primary Key | `step_id` |
| 주요 참조(FK) | `iq_item_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 644 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 645 | 시험 개정 ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | iq_item.iq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 646 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 647 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 648 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 649 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 650 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 651 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_step_1` | UNIQUE | `iq_item_id, step_order` | UNIQUE(iq_item_id, step_order) |
| `uq_iq_step_1_rule` | 업무 검증 | `iq_item_id, step_order` | step_order >= 1 |
| `rule_iq_step_2` | 업무 검증 | - | 상위 프로토콜이 APPROVED이면 절차 구조·본문을 변경하지 않는다. 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 iq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-iq_execution"></a>
### 41. IQ 시험 수행 (`iq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `iq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 652 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 653 | 수행 프로토콜 개정 ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | iq_item.iq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 654 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 655 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 656 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. | `TRUE` |
| 657 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 658 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA. 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 659 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 660 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 661 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 662 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 663 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 664 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 프로토콜 승인 상태와 독립적으로 관리. | `DRAFT` |
| 665 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 666 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 667 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 668 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 669 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 670 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 671 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 672 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 673 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 674 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_execution_1` | UNIQUE | `iq_item_id, attempt_no, record_revision` | UNIQUE(iq_item_id, attempt_no, record_revision) |
| `uq_iq_execution_1_rule` | 업무 검증 | `iq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개 |
| `rule_iq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_iq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수이며 해당 execution_id를 원본으로 deviation을 연결한다. |
| `rule_iq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_iq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록 후에 가능하다. 연결된 일탈의 필요한 승인·종결 상태도 검사한다. |
| `ck_iq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 IQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-iq_step_execution"></a>
### 42. IQ 절차 수행 (`iq_step_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 수행 회차의 절차 완료 여부 및 절차 증적 연결 단위 |
| Primary Key | `step_execution_id` |
| 주요 참조(FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 675 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 676 | 수행 ID | `execution_id` | `uuid` | N | Y | `iq_execution.execution_id` | Y | - | N | Y | N | Y | iq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 677 | 절차 ID | `step_id` | `uuid` | N | Y | `iq_step.step_id` | Y | - | N | Y | N | Y | iq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 678 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 679 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 680 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 681 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 682 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 683 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 684 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_iq_step_execution_1_rule` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_iq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 IQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## OQ

<a id="table-oq_assessment"></a>
### 43. 운전 적격성 평가 (`oq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 OQ 시험 묶음 및 문서 개정 정보. 프로토콜과 수행 결과의 승인 상태를 구분한다. |
| Primary Key | `oq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 685 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 686 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 687 | OQ 번호 | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `OQ-VP-SYS-010-20260529` |
| 688 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 운전 적격성 평가 문서 제목 | `Operational Qualification` |
| 689 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 690 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 691 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | OQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 692 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 OQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 693 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 694 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 695 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 696 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 697 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 698 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | OQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 699 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 700 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"oq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_assessment_1` | UNIQUE | `project_id, oq_no, revision_number` | UNIQUE(project_id, oq_no, revision_number) |
| `uq_oq_assessment_1_rule` | 업무 검증 | `project_id, oq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_oq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_oq_assessment_current` | UNIQUE | `project_id, oq_no` | UNIQUE (project_id, oq_no) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 oq_item의 실제 개정이며 같은 프로젝트·OQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 oq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다.

[↑ 맨 위로](#top)

---

<a id="table-oq_item"></a>
### 44. OQ 상세 테스트 항목 (`oq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `oq_item_id` |
| 주요 참조(FK) | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 701 | OQ 항목 ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 702 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | N | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 703 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 704 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `OQ-AT-L01` |
| 705 | 테스트 케이스 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. 별도 분류값이 아님. | `Audit Trail` |
| 706 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `사용자 데이터 변경 시 감사추적 자동 생성` |
| 707 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `변경 전/후 값, 사용자, 날짜/시간, IP 기록` |
| 708 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 이 시험 개정의 독립적인 승인 상태이며 상위 문서 상태로 덮어쓰지 않음. | `DRAFT` |
| 709 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 710 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 711 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 712 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 713 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 714 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 715 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 716 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 717 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 718 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 719 | 등록 출처 | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), PACKAGE(시스템 패키지), AI(AI 초안). | `MANUAL` |
| 720 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 721 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `IQ-CORE` |
| 722 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 723 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `IQ-PKG-001` |
| 724 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 725 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 726 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 727 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 권한자가 사유와 대상을 확인하여 수행한 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 728 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 729 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 730 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 화면 폐기 완료 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_oq_item_1_rule` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 OQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_oq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 승인 완료된 프로토콜과 절차는 직접 수정하지 않고 새 개정 행을 생성한다. |
| `rule_oq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_oq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_oq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → OQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_oq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_oq_item_7` | 업무 검증 | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | is_disposed = TRUE이면 disposal_reason, disposal_signature_id, disposed_by, disposed_at 필수. 폐기 서명의 실제 대상 테이블·PK는 이 시험 개정이고 서명자·시각은 disposed_by / disposed_at과 일치해야 한다. |
| `rule_oq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_oq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 oq_step, 실제 결과·서명·재수행 이력은 oq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-oq_step"></a>
### 45. OQ 시험 절차 (`oq_step`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 프로토콜 개정에 포함된 순서 있는 세부 절차 |
| Primary Key | `step_id` |
| 주요 참조(FK) | `oq_item_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 731 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 732 | 시험 개정 ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | oq_item.oq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 733 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 734 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 735 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 736 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 737 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 738 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_step_1` | UNIQUE | `oq_item_id, step_order` | UNIQUE(oq_item_id, step_order) |
| `uq_oq_step_1_rule` | 업무 검증 | `oq_item_id, step_order` | step_order >= 1 |
| `rule_oq_step_2` | 업무 검증 | - | 상위 프로토콜이 APPROVED이면 절차 구조·본문을 변경하지 않는다. 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 oq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-oq_execution"></a>
### 46. OQ 시험 수행 (`oq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `oq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 739 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 740 | 수행 프로토콜 개정 ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | oq_item.oq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 741 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 742 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 743 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. | `TRUE` |
| 744 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 745 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA. 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 746 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 747 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 748 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 749 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 750 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 751 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 프로토콜 승인 상태와 독립적으로 관리. | `DRAFT` |
| 752 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 753 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 754 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 755 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 756 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 757 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 758 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 759 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 760 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 761 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_execution_1` | UNIQUE | `oq_item_id, attempt_no, record_revision` | UNIQUE(oq_item_id, attempt_no, record_revision) |
| `uq_oq_execution_1_rule` | 업무 검증 | `oq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개 |
| `rule_oq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_oq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수이며 해당 execution_id를 원본으로 deviation을 연결한다. |
| `rule_oq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_oq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록 후에 가능하다. 연결된 일탈의 필요한 승인·종결 상태도 검사한다. |
| `ck_oq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 OQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-oq_step_execution"></a>
### 47. OQ 절차 수행 (`oq_step_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 수행 회차의 절차 완료 여부 및 절차 증적 연결 단위 |
| Primary Key | `step_execution_id` |
| 주요 참조(FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 762 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 763 | 수행 ID | `execution_id` | `uuid` | N | Y | `oq_execution.execution_id` | Y | - | N | Y | N | Y | oq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 764 | 절차 ID | `step_id` | `uuid` | N | Y | `oq_step.step_id` | Y | - | N | Y | N | Y | oq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 765 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 766 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 767 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 768 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 769 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 770 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 771 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_oq_step_execution_1_rule` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_oq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 OQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## PQ

<a id="table-pq_assessment"></a>
### 48. 성능 적격성 평가 (`pq_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 PQ 시험 묶음 및 문서 개정 정보. 프로토콜과 수행 결과의 승인 상태를 구분한다. |
| Primary Key | `pq_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 772 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 773 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 774 | PQ 번호 | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `PQ-VP-SYS-008-20260422` |
| 775 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 성능 적격성 평가 문서 제목 | `Performance Qualification` |
| 776 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 777 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 778 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | PQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 779 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 PQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 780 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 781 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 개별 항목/수행의 상태를 집계하며 하위 행에 일괄 덮어쓰지 않음. | `DRAFT` |
| 782 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 783 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 784 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 785 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | PQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 786 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 787 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"pq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_assessment_1` | UNIQUE | `project_id, pq_no, revision_number` | UNIQUE(project_id, pq_no, revision_number) |
| `uq_pq_assessment_1_rule` | 업무 검증 | `project_id, pq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_pq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_pq_assessment_current` | UNIQUE | `project_id, pq_no` | UNIQUE (project_id, pq_no) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 pq_item의 실제 개정이며 같은 프로젝트·PQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 pq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다.

[↑ 맨 위로](#top)

---

<a id="table-pq_item"></a>
### 49. PQ 상세 테스트 항목 (`pq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `pq_item_id` |
| 주요 참조(FK) | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 788 | PQ 항목 ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 789 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | N | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 790 | 시험 표시 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `PQ-NEW-01` |
| 791 | 시험 항목명 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. | `업무 시나리오 검증` |
| 792 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 793 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | PQ 테스트 검증 수행 상세 절차 | `연속 3배치 이상 생산 공정 정상 완료 검증` |
| 794 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `모든 배치가 사양에 맞게 정상 생산 완료되어야 함` |
| 795 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 796 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 797 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 798 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 799 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 800 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 801 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 802 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 803 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 804 | 프로토콜 승인 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려) | `DRAFT` |
| 805 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 806 | 등록 출처 | `source_type` | `varchar(20)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), PACKAGE(시스템 패키지), AI(AI 초안). | `MANUAL` |
| 807 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 808 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `IQ-CORE` |
| 809 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 810 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `IQ-PKG-001` |
| 811 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 812 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 813 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 814 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 권한자가 사유와 대상을 확인하여 수행한 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 815 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 816 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 817 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 화면 폐기 완료 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `uq_pq_item_1_rule` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 PQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_pq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 승인 완료된 프로토콜과 절차는 직접 수정하지 않고 새 개정 행을 생성한다. |
| `rule_pq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_pq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_pq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → PQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_pq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_pq_item_7` | 업무 검증 | `disposal_reason, disposal_signature_id, is_disposed, disposed_by, disposed_at` | is_disposed = TRUE이면 disposal_reason, disposal_signature_id, disposed_by, disposed_at 필수. 폐기 서명의 실제 대상 테이블·PK는 이 시험 개정이고 서명자·시각은 disposed_by / disposed_at과 일치해야 한다. |
| `rule_pq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_pq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 pq_step, 실제 결과·서명·재수행 이력은 pq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트 및 업무 유형에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·업무 유형의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-pq_step"></a>
### 50. PQ 시험 절차 (`pq_step`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 프로토콜 개정에 포함된 순서 있는 세부 절차 |
| Primary Key | `step_id` |
| 주요 참조(FK) | `pq_item_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 818 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 819 | 시험 개정 ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | pq_item.pq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 820 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 821 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 822 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 823 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 824 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 825 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_step_1` | UNIQUE | `pq_item_id, step_order` | UNIQUE(pq_item_id, step_order) |
| `uq_pq_step_1_rule` | 업무 검증 | `pq_item_id, step_order` | step_order >= 1 |
| `rule_pq_step_2` | 업무 검증 | - | 상위 프로토콜이 APPROVED이면 절차 구조·본문을 변경하지 않는다. 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 pq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-pq_execution"></a>
### 51. PQ 시험 수행 (`pq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `pq_item_id, executed_by, execution_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 826 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 827 | 수행 프로토콜 개정 ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | pq_item.pq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 828 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 829 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 830 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. | `TRUE` |
| 831 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 832 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA. 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 833 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 834 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 835 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 836 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 837 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 838 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 프로토콜 승인 상태와 독립적으로 관리. | `DRAFT` |
| 839 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 840 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 841 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 842 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 843 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 844 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 845 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 846 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 847 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 848 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_execution_1` | UNIQUE | `pq_item_id, attempt_no, record_revision` | UNIQUE(pq_item_id, attempt_no, record_revision) |
| `uq_pq_execution_1_rule` | 업무 검증 | `pq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개 |
| `rule_pq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_pq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수이며 해당 execution_id를 원본으로 deviation을 연결한다. |
| `rule_pq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_pq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록 후에 가능하다. 연결된 일탈의 필요한 승인·종결 상태도 검사한다. |
| `ck_pq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 PQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-pq_step_execution"></a>
### 52. PQ 절차 수행 (`pq_step_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 수행 회차의 절차 완료 여부 및 절차 증적 연결 단위 |
| Primary Key | `step_execution_id` |
| 주요 참조(FK) | `execution_id, step_id, confirmed_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 849 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 850 | 수행 ID | `execution_id` | `uuid` | N | Y | `pq_execution.execution_id` | Y | - | N | Y | N | Y | pq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 851 | 절차 ID | `step_id` | `uuid` | N | Y | `pq_step.step_id` | Y | - | N | Y | N | Y | pq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 852 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 853 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 854 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 855 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 856 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 857 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 858 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `uq_pq_step_execution_1_rule` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_pq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 PQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## VSR

<a id="table-vsr_assessment"></a>
### 53. 밸리데이션 종합 보고서 (`vsr_assessment`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트별 최종 밸리데이션 결론과 종합 보고서 관리 |
| Primary Key | `vsr_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 859 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 860 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | N | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 861 | VSR 번호 | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR 문서 번호. 개정 간 동일 번호를 유지. | `VSR-VP-SYS-008-20260422` |
| 862 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 밸리데이션 종합 보고서 제목 | `Validation Summary Report` |
| 863 | 밸리데이션 결론 | `overall_conclusion` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 밸리데이션 종합 결론. 선택 항목이며 업무 승인 상태와 구분한다. | - |
| 864 | 결론 상세 설명 | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | 밸리데이션 결론의 상세 설명과 조건사항. NULL을 허용한다. | `OQ-GMP-02 일탈 해결 완료 후 최종 승인 가능` |
| 865 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 866 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 867 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | VSR 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 868 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 VSR 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 869 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | VSR 업무 확인 상태. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 별도 생성 산출물의 승인 상태와 구분. | `DRAFT` |
| 870 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 871 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 872 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 873 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | VSR 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 874 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 875 | 원본 기준값 | `source_fingerprint` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 선택 활동과 원본 개정 참조·요약값으로 계산한 SHA-256. 변경 시 재확인 대상 판별. | - |
| 876 | 집계 시각 | `captured_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 현재 VSR 활동 요약을 생성한 시각. | `2026-09-01T00:00:00Z` |
| 877 | VSR 확인 워크플로우 | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | VSR 업무 확인 요청·검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 878 | VSR 확인 서명 | `approval_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | VSR 최종 확인 완료 시 전자서명. | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vsr_assessment_1` | UNIQUE | `project_id, vsr_no, revision_number` | UNIQUE(project_id, vsr_no, revision_number) |
| `uq_vsr_assessment_1_rule` | 업무 검증 | `project_id, vsr_no, revision_number` | 같은 문서의 최신 개정은 최대 1개 |
| `rule_vsr_assessment_2` | 업무 검증 | - | 승인 완료된 VSR 헤더와 하위 요약값은 보존한다. 원본 변경 시 새 VSR 개정에 DRAFT로 재집계하여 확인을 받는다. |
| `rule_vsr_assessment_3` | 업무 검증 | - | VSR에는 프로젝트에서 선택한 수행 단계만 집계하며 RTM은 제외한다. VSR 자체 확인은 헤더에 표시하고 상세 요약에 재귀적으로 저장하지 않는다. |

#### 업무 규칙

화면 활동 요약의 원본 상태 확인용 기록이다. 보고서 본문·PDF 생성 및 해당 문서 승인은 deliverable_document / deliverable_revision에서 별도로 관리한다.

[↑ 맨 위로](#top)

---

<a id="table-vsr_item"></a>
### 54. VSR 활동 요약 항목 (`vsr_item`)

| 항목 | 정의 |
|---|---|
| 설명 | 밸리데이션 활동별 문서, 결과, 일탈 및 승인 정보 요약 관리 |
| Primary Key | `vsr_item_id` |
| 주요 참조(FK) | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 879 | VSR 항목 ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR 활동 요약 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 880 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | N | N | Y | 상위 VSR 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 881 | 활동 구분 | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 집계 활동 코드: VP, VA, QIA, URS, FDS_GROUP(화면 F&DS), FRA, DQ, IQ, OQ, PQ. RTM 제외; VSR 자기 상태는 헤더에서 표시. | `IQ` |
| 882 | 문서 번호 | `doc_no` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 집계 당시 표시한 해당 활동 문서 번호. 미승인·미생성 시 NULL. | `VP-SYS-008-20260422 · URS` |
| 883 | 개정 차수 | `revision_no` | `varchar(20)` | N | N | - | N | `'1'` | N | N | N | Y | 집계 당시 표시한 원본 승인 버전. 특정 개정 식별은 source_revision_refs에 보존. | `ver1` |
| 884 | 일탈 건수 | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 집계 당시 일탈 건수·미결 현황 표시 스냅샷. 독립적으로 수동 편집하지 않음. | - |
| 885 | 결론/상태 | `item_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | 집계 당시 활동 승인 상태: DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 원본 업무 상태를 자동 조회한 스냅샷. | `APPROVED` |
| 886 | 승인자명 | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | Y | Y | 원본 승인 당시 승인자 표시명 스냅샷. | `홍길동` |
| 887 | 승인일 | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | 원본 승인 당시 화면 표시용 승인일 스냅샷. 정확한 서명 시각은 원본 기록에서 조회. | `2024-02-20` |
| 888 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 889 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 890 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 891 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | N | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 892 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 893 | 프로젝트 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | 해당 VSR과 같은 프로젝트의 선택된 수행 활동. | `00000000-0000-0000-0000-000000000001` |
| 894 | 산출물 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | 산출물 문서가 생성된 경우 참조하는 개정. 업무 항목 승인만 있는 단계에서는 NULL 가능. | `00000000-0000-0000-0000-000000000001` |
| 895 | 원본 개정 참조 목록 | `source_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | 집계 근거인 정확한 업무 개정 목록: [{table_name, record_id, version}]. table_name은 실제 테이블명이며 IQ/OQ/PQ 결과는 iq_execution / oq_execution / pq_execution의 execution_id, FDS/DDS는 fds_spec.fds_id / dds_spec.dds_id 개정 PK 사용. 미승인 단계는 빈 배열. | `[]` |
| 896 | 원본 요약 스냅샷 | `source_snapshot` | `jsonb` | N | N | - | Y | `'{}'::jsonb` | N | N | Y | Y | 집계 당시 승인 표의 columns / rows와 승인정보. 원본 개정 참조와 함께 표시값을 고정하며 미승인 단계는 빈 객체. | `{}` |
| 897 | 활동 원본 기준값 | `source_fingerprint` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | 해당 단계의 원본 참조와 스냅샷을 해시한 값. 미승인 원본이 없으면 NULL. | - |
| 898 | 활동 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 프로젝트의 선택 활동 순서. | `1` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vsr_item_1` | UNIQUE | `vsr_id, project_activity_id` | UNIQUE(vsr_id, project_activity_id) |
| `uq_vsr_item_1_rule` | 업무 검증 | `vsr_id, project_activity_id` | activity_code와 project_activity의 활동 코드는 일치해야 한다 |
| `rule_vsr_item_2` | 업무 검증 | `project_activity_id, document_revision_id` | project_activity_id / document_revision_id / source_revision_refs는 같은 프로젝트를 가리켜야 한다. 참조 원본의 존재·유형·개정과 승인 여부를 검증한다. |
| `rule_vsr_item_3` | 업무 검증 | `item_status` | item_status = APPROVED이면 source_revision_refs가 비어 있지 않아야 하며 승인 시점의 source_snapshot과 표시값을 보존한다. |
| `rule_vsr_item_4` | 업무 검증 | - | 일탈 표시값은 해당 활동의 일탈 기록에서 집계한다. |
| `rule_vsr_item_5` | 업무 검증 | - | 상위 VSR 승인 후 상세행은 수정하지 않는다. 새 집계는 새 VSR 개정의 상세행으로 기록한다. |

#### 업무 규칙

source_revision_refs는 실제 테이블명·개정 PK·버전의 다형 참조 목록으로 물리 FK가 아니다. deliverable_revision.source_refs와 동일한 객체 구조를 사용한다. 문서 파일 개정과 업무 확인 상태를 같은 것으로 취급하지 않는다.

[↑ 맨 위로](#top)

---

## Workflow

<a id="table-workflow_instance"></a>
### 55. Workflow 인스턴스 (`workflow_instance`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서별 검토·승인 Workflow 진행 정보 관리 |
| Primary Key | `workflow_instance_id` |
| 주요 참조(FK) | `requested_by, created_by, updated_by, project_id, workflow_config_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 899 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서별 검토·승인 Workflow 고유 식별자 | `UUID` |
| 900 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 검토·승인 대상 테이블명. 업무 항목/문서 외에 system_asset, project_closure_request 등도 허용한다. | `fds_spec` |
| 901 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | 대상 테이블 행 PK. 다형 참조이므로 실제 FK가 아니며 대상 존재·프로젝트·버전을 함께 검증한다. | `UUID` |
| 902 | 대상 문서 버전 | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 승인 대상의 표시 버전 또는 개정번호. 인벤토리는 system_asset.revision_number, 종료 요청은 CLOSE-{request_version}에 대응 | `v1.0` |
| 903 | 상신자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 문서를 검토·승인 절차에 상신한 사용자 | `UUID` |
| 904 | 상신 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 문서가 검토·승인 절차에 상신된 시각 | `2026-08-31T15:00:00Z` |
| 905 | Workflow 상태 | `workflow_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | 전체 Workflow 상태. DRAFT, IN_PROGRESS, APPROVED, REJECTED, CANCELLED | `IN_PROGRESS` |
| 906 | 현재 단계 순서 | `current_step_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 현재 처리 중인 검토·승인 단계 순서. 상신 시 최초 비삭제 단계의 step_order를 저장하며 1로 고정하지 않는다 | `1` |
| 907 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 승인·반려 또는 취소로 Workflow가 종료된 시각 | `2026-08-31T17:00:00Z` |
| 908 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00Z` |
| 909 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 레코드 생성자 | `UUID` |
| 910 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00Z` |
| 911 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 레코드 최종 수정자 | `UUID` |
| 912 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 913 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 업무에 대한 결재이면 필수. 인벤토리 등 프로젝트 외 결재는 NULL | `00000000-0000-0000-0000-000000000001` |
| 914 | 승인 구분 | `approval_scope` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ITEM/PROTOCOL/RESULT/DOCUMENT/INVENTORY/CLOSURE/DISPOSAL/DEVIATION_ACTION/DEVIATION_COMPLETION/DEVIATION_CLOSE | `DOCUMENT` |
| 915 | 적용 결재선 설정 ID | `workflow_config_id` | `uuid` | N | Y | `project_workflow_config.config_id` | N | - | N | Y | N | Y | 상신 때 사용한 프로젝트 결재선 설정. 인벤토리 자체 경로 등은 NULL | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_instance_1` | UNIQUE | `target_table_name, target_record_id, target_version, approval_scope, workflow_status` | UNIQUE (target_table_name, target_record_id, target_version, approval_scope) WHERE workflow_status IN ('DRAFT','IN_PROGRESS') |

#### 업무 규칙

상신할 때 설정의 단계·모드·주/대체 담당자를 workflow_step 및 workflow_step_assignee로 복사한다. 이후 기본 설정 변경이 이미 진행 중인 결재선을 바꾸지 않는다. 작성자·검토자·승인자와 대체자는 서로 다른 활성 계정으로 지정하고, 인벤토리도 작성·검토·승인자가 서로 다른지 검사한다.

하나의 시험 항목이라도 PROTOCOL 승인과 수행 RESULT 승인을 분리한다. 승인 범위·대상 테이블·PK·버전을 함께 식별한다. 작성자는 requested_by 및 SUBMIT 처리 이력으로 기록한다.

DEVIATION_ACTION은 deviation_action_round의 PK와 ACTION-{round_number}-{revision_number}를 대상으로 한다. DEVIATION_COMPLETION과 DEVIATION_CLOSE는 deviation의 PK 및 전자서명에 정의한 완료·종료 버전을 사용한다. 반려·취소된 조치 내용을 수정하여 재상신할 때 새 조치 개정과 Workflow를 생성한다.

[↑ 맨 위로](#top)

---

<a id="table-workflow_step"></a>
### 56. Workflow 단계 (`workflow_step`)

| 항목 | 정의 |
|---|---|
| 설명 | Workflow 내 단계별 처리 상태 및 담당자 관리 |
| Primary Key | `workflow_step_id` |
| 주요 참조(FK) | `workflow_instance_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 916 | Workflow 단계 ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Workflow 검토·승인 단계 고유 식별자. deleted_at IS NULL인 행에 (workflow_instance_id, step_order) 중복을 허용하지 않는다. | `UUID` |
| 917 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | 단계가 소속된 Workflow 식별자 | `UUID` |
| 918 | 단계 순서 | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | 동일 Workflow 내 검토·승인 처리 순서 | `1` |
| 919 | 단계 유형 | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 처리 단계 유형. REVIEW 또는 APPROVE. SUBMIT/CANCEL은 단계 유형이 아닌 처리 이력의 action_type으로 기록한다 | `REVIEW` |
| 920 | 단계명 | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 화면에 표시할 검토·승인 단계명 | `품질 검토` |
| 921 | 처리 모드 | `execution_mode` | `varchar(20)` | N | N | - | Y | `'SERIAL'` | N | N | N | Y | SERIAL=직렬, PARALLEL=병렬. 동일 단계의 여러 담당자 처리 순서 | `SERIAL` |
| 922 | 단계 상태 | `step_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | 단계 상태. PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 923 | 처리 기한 | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계의 검토·승인 처리 예정 기한 | `2026-09-02T18:00:00Z` |
| 924 | 처리 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계가 승인·반려 등으로 완료된 시각 | `2026-09-01T10:00:00Z` |
| 925 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00Z` |
| 926 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 단계 생성자 | `UUID` |
| 927 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00Z` |
| 928 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 단계 최종 수정자 | `UUID` |
| 929 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_step_1` | UNIQUE | `workflow_instance_id, step_order, deleted_at` | UNIQUE (workflow_instance_id, step_order) WHERE deleted_at IS NULL |
| `ck_workflow_step_2` | CHECK | `execution_mode` | CHECK (execution_mode IN ('SERIAL','PARALLEL')) |
| `ck_workflow_step_3` | CHECK | `step_order` | CHECK (step_order >= 1) |

#### 업무 규칙

주 담당자·대체자 및 담당자 순서는 workflow_step_assignee에 저장한다. step_order는 단계 순서이며 execution_mode는 같은 단계 안의 담당자 처리 방식을 구분한다.

병렬 단계도 지정된 담당자들의 처리가 모두 완료되어야 다음 단계로 진행한다. 검토 단계는 REVIEW, 승인 단계는 APPROVE 처리 이력을 남긴다.

[↑ 맨 위로](#top)

---

<a id="table-workflow_step_assignee"></a>
### 57. 결재 단계 담당자 (`workflow_step_assignee`)

| 항목 | 정의 |
|---|---|
| 설명 | 검토·승인 단계의 복수 주 담당자와 대체 담당자 |
| Primary Key | `assignment_id` |
| 주요 참조(FK) | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 930 | 담당 배정 ID | `assignment_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 931 | 결재 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | workflow_step.workflow_step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 932 | 주 담당자 ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 933 | 대체 담당자 ID | `substitute_user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 934 | 담당자 순서 | `assignee_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 직렬 단계에서 처리할 순서 | `1` |
| 935 | 처리 상태 | `assignment_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | N | N | Y | PENDING/IN_PROGRESS/APPROVED/REJECTED/SKIPPED | `PENDING` |
| 936 | 처리 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 처리 완료 시각 값 | `2026-09-01T00:00:00Z` |
| 937 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 938 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 939 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 940 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_step_assignee_1` | UNIQUE | `workflow_step_id, assignee_order` | UNIQUE (workflow_step_id, assignee_order) |
| `uq_workflow_step_assignee_2` | UNIQUE | `workflow_step_id, assignee_id` | UNIQUE (workflow_step_id, assignee_id) |
| `ck_workflow_step_assignee_3` | CHECK | `assignee_order` | CHECK (assignee_order >= 1) |
| `rule_workflow_step_assignee_4` | 업무 검증 | - | 주 담당자와 대체 담당자는 같을 수 없다. |

#### 업무 규칙

실제 처리자는 approval_action.actor_id로 기록한다. 주 담당자 또는 권한이 있는 대체 담당자 중 한 명이 같은 배정을 처리한다.

직렬은 앞선 배정 완료 후, 병렬은 현재 단계의 미완료 배정이 처리 가능하다.

[↑ 맨 위로](#top)

---

<a id="table-approval_action"></a>
### 58. 승인 처리 이력 (`approval_action`)

| 항목 | 정의 |
|---|---|
| 설명 | 검토·승인·반려 등 단계별 실제 처리 이력 관리 |
| Primary Key | `approval_action_id` |
| 주요 참조(FK) | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 941 | 승인 처리 이력 ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 검토·승인 처리 이력 고유 식별자 | `UUID` |
| 942 | Workflow 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | 처리 이력이 귀속되는 Workflow 단계. SUBMIT은 최초 비삭제 단계, CANCEL은 진행 중인 현재 단계(시작 전이면 최초 단계), REVIEW/APPROVE/REJECT는 실제 처리 단계에 연결한다. 귀속 단계가 없으면 이력을 생성하지 않는다. | `UUID` |
| 943 | 처리 담당 배정 ID | `workflow_step_assignee_id` | `uuid` | N | Y | `workflow_step_assignee.assignment_id` | N | - | N | Y | N | Y | REVIEW/APPROVE/REJECT 처리 배정. SUBMIT/CANCEL은 NULL | `00000000-0000-0000-0000-000000000001` |
| 944 | 처리 유형 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 수행한 처리 유형. SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 945 | 처리자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 실제 검토·승인·반려 처리를 수행한 사용자 | `UUID` |
| 946 | 처리 의견 | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | 검토·승인 처리 시 입력한 의견 | `검토 결과 이상 없음` |
| 947 | 반려 사유 | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | REJECT 처리 시 입력하는 반려 사유 | `증적 파일 보완 필요` |
| 948 | 전자서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 처리에 연결된 전자서명 기록. 서명이 있으면 signer_id는 actor_id와 같고 대상 테이블·레코드·버전은 Workflow 대상과 일치해야 한다. SUBMIT/REVIEW/APPROVE/REJECT의 서명 동작은 action_type과 같아야 한다. CANCEL은 서명 없이 취소 사유와 Audit Trail을 기록한다. | `UUID` |
| 949 | 처리 시각 | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 상신·검토·승인·반려가 실제 처리된 시각 | `2026-09-01T10:00:00Z` |
| 950 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-09-01T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_approval_action_1` | 업무 검증 | - | REVIEW/APPROVE/REJECT일 때 workflow_step_assignee_id는 필수이다. SUBMIT/CANCEL에서는 NULL이다. |
| `rule_approval_action_2` | 업무 검증 | - | 한 담당 배정에 유효한 REVIEW/APPROVE/REJECT 처리 이력은 한 건만 존재한다. |
| `rule_approval_action_3` | 업무 검증 | - | SUBMIT/REVIEW/APPROVE/REJECT는 전자서명이 필수이다. CANCEL은 전자서명 없이 취소 사유와 감사기록을 남긴다. |
| `uq_approval_assignee_action` | UNIQUE | `workflow_step_assignee_id, action_type` | UNIQUE (workflow_step_assignee_id) WHERE action_type IN ('REVIEW','APPROVE','REJECT') |

#### 업무 규칙

담당 배정의 workflow_step_id와 이력의 workflow_step_id는 같아야 한다. 실제 처리자는 주 담당자 또는 대체 담당자이며 전자서명 signer_id와 일치한다.

SUBMIT은 최초 단계, CANCEL은 현재 단계(시작 전이면 최초 단계)에 기록한다. 서명 대상 테이블·PK·버전 및 동작은 workflow_instance와 action_type에 일치해야 한다.

동일 배정의 반려를 승인으로 덮어쓰지 않는다. 재상신 시 새 Workflow 또는 새 단계·담당 배정을 생성한다. 처리 대상 단계와 배정을 잠근 뒤 현재 진행 상태·미처리 여부·대체자 자격을 다시 검사하고 서명·처리 이력·단계 및 전체 상태 전환을 원자적으로 저장한다.

[↑ 맨 위로](#top)

---

<a id="table-project_workflow_config"></a>
### 59. 프로젝트 결재선 설정 (`project_workflow_config`)

| 항목 | 정의 |
|---|---|
| 설명 | 프로젝트 기본 및 활동별 작성·검토·승인 결재선 설정 |
| Primary Key | `config_id` |
| 주요 참조(FK) | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 951 | 결재선 설정 ID | `config_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 952 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 953 | 수행 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | N | - | N | Y | N | Y | 활동별 설정일 때 지정. 프로젝트 기본 설정은 NULL | `00000000-0000-0000-0000-000000000001` |
| 954 | 설정 버전 | `config_version` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 적용 후 변경할 때 증가하는 설정판 | `1` |
| 955 | 결재선 구성 | `route_definition` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 작성자 단계(author_stage), 검토 단계 배열(review_stages), 승인 단계 배열(approval_stages). 각 단계에 순서·SERIAL/PARALLEL·주/대체 담당자 배열 저장 | - |
| 956 | 적용 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 현재 적용 중인 설정 여부. 새 설정 적용 시 이전 행은 FALSE로 전환하고 이전 적용 서명·시각·원문은 보존한다. | `FALSE` |
| 957 | 적용 서명 ID | `applied_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | electronic_signature.signature_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 958 | 적용 시각 | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 적용 시각 값 | `2026-09-01T00:00:00Z` |
| 959 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 960 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 961 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 962 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_project_workflow_config_1` | 업무 검증 | - | 활동별 설정의 project_activity_id는 동일 project_id에 속해야 한다. |
| `rule_project_workflow_config_2` | 업무 검증 | `project_id, project_activity_id, config_version` | 프로젝트 기본 설정은 (project_id, config_version), 활동별 설정은 (project_activity_id, config_version)가 유일하다. 동일 프로젝트/활동의 현재 적용 설정은 각각 최대 한 건이다. |
| `rule_project_workflow_config_3` | 업무 검증 | `is_applied` | is_applied=TRUE일 때 applied_signature_id와 applied_at은 필수이다. |
| `uq_workflow_config_project_version` | UNIQUE | `project_id, config_version, project_activity_id` | UNIQUE (project_id, config_version) WHERE project_activity_id IS NULL |
| `uq_workflow_config_activity_version` | UNIQUE | `project_activity_id, config_version` | UNIQUE (project_activity_id, config_version) WHERE project_activity_id IS NOT NULL |
| `uq_workflow_config_project_applied` | UNIQUE | `project_id, project_activity_id, is_applied` | UNIQUE (project_id) WHERE project_activity_id IS NULL AND is_applied=TRUE |
| `uq_workflow_config_activity_applied` | UNIQUE | `project_activity_id, is_applied` | UNIQUE (project_activity_id) WHERE project_activity_id IS NOT NULL AND is_applied=TRUE |

#### 업무 규칙

활동별 결재선이 있으면 사용하고 없으면 프로젝트 기본 결재선을 사용한다. RTM에 대한 수행 단계 설정은 만들지 않는다.

route_definition의 단계는 {step_order, execution_mode, assignees:[{user_id, substitute_user_id, assignee_order}]} 구조다. JSON 내부 사용자 식별자는 물리 FK가 아니므로 저장 시 존재·권한·중복을 검사한다. 작성자·검토자·승인자·대체 담당자는 프로젝트 권한이 있는 서로 다른 활성 개인 계정이어야 한다.

작성자 단계는 작성자 후보 지정이다. 실제 상신자는 workflow_instance.requested_by로 기록한다. 실행 결재는 workflow_step 및 workflow_step_assignee에 복사한다.

적용 설정 전환은 프로젝트를 잠그고 수행한다. 이전 is_applied 해제와 새 설정 적용은 같은 트랜잭션으로 저장한다. is_applied는 관리 상태로서 원래 CONFIG_APPLY 서명 원문에 포함하지 않는다. 이미 적용한 route_definition·config_version·서명·적용 시각은 변경하지 않으며, 활동별 설정은 같은 프로젝트의 project_activity를 참조해야 한다.

[↑ 맨 위로](#top)

---

## Traceability

<a id="table-traceability_link"></a>
### 60. 공통 추적 관계 (`traceability_link`)

| 항목 | 정의 |
|---|---|
| 설명 | 화면에서 선택한 URS·설계 문서·DQ·FRA·시험 항목 간 연결과 대시보드 RTM의 원본 관계 관리 |
| Primary Key | `traceability_link_id` |
| 주요 참조(FK) | `project_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 963 | 추적 관계 ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ 항목 간 추적 관계 고유 식별자. 삭제되지 않은 데이터는 (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) 조합의 중복을 허용하지 않는다. | `UUID` |
| 964 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 추적 관계가 속한 프로젝트 ID | `UUID` |
| 965 | 출발 엔터티 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 연결 대상 유형: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. 각 유형의 특정 개정 행 PK를 연결한다. | `REQUIREMENT` |
| 966 | 출발 엔터티 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 유형에 해당하는 테이블의 특정 개정 행 PK. 물리 FK가 아닌 다형 참조; 논리 key나 표시 번호로 대체하지 않는다. | `UUID` |
| 967 | 연결 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 연결 대상 유형: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. 각 유형의 특정 개정 행 PK를 연결한다. | `OQ_ITEM` |
| 968 | 연결 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 유형에 해당하는 테이블의 특정 개정 행 PK. 물리 FK가 아닌 다형 참조; 논리 key나 표시 번호로 대체하지 않는다. | `UUID` |
| 969 | 관계 유형 | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 970 | 연결 근거 | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | 두 산출물 항목을 연결한 업무적 근거 | `URS 요구사항을 IQ 시험으로 검증` |
| 971 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 972 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 생성한 사용자 ID | `UUID` |
| 973 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 974 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 최종 수정한 사용자 ID | `UUID` |
| 975 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 추적 관계 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_traceability_link_1` | UNIQUE | `project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type` | UNIQUE(project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) |
| `rule_traceability_link_1_rule` | 업무 검증 | `project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type` | 삭제되지 않은 행에 |
| `rule_traceability_link_2` | 업무 검증 | - | 연결 양 끝은 동일 프로젝트에 속하고 등록된 대상 유형 및 실제 개정 행으로 존재해야 한다. |
| `rule_traceability_link_3` | 업무 검증 | - | REQUIREMENT → IQ_ITEM / OQ_ITEM / PQ_ITEM의 VERIFIED_BY는 하나의 시험에 복수 URS를 연결하는 화면 관계다. |
| `rule_traceability_link_4` | 업무 검증 | - | FDS_SPEC / DDS_SPEC는 fds_spec.fds_id / dds_spec.dds_id를 의미한다. |
| `rule_traceability_link_5` | 업무 검증 | - | 대시보드 RTM은 연결 관계와 현재 유효한 원본·수행 상태를 조회한다. RTM 전용 문서·시험·승인 상태를 추가로 생성하지 않는다. |
| `rule_traceability_link_6` | 업무 검증 | - | 연결을 개정할 때 과거 승인 대상의 원본 관계를 삭제·덮어쓰지 않는다. 새 개정에 새 관계를 만들고 과거 링크를 보존한다. |

#### 업무 규칙

커버리지는 URS와 시험의 연결 여부로 집계하며 시험 PASS 또는 승인완료 비율과 구분한다. FDS·DDS 문서의 직접 URS 연결은 파일 승인 화면의 선택 관계에 한해 저장한다.

[↑ 맨 위로](#top)

---

## Deviation

<a id="table-deviation"></a>
### 61. 일탈 관리 (`deviation`)

| 항목 | 정의 |
|---|---|
| 설명 | 시험 FAIL에 따른 사유·즉시 조치, 재수행, 시정·예방 조치, 완료보고 및 종료 처리 관리 |
| Primary Key | `deviation_id` |
| 주요 참조(FK) | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 976 | 일탈 ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 일탈 고유 식별자 | `UUID` |
| 977 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 일탈이 발생한 Validation 프로젝트 | `UUID` |
| 978 | 발생 대상 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 실패한 원본 수행의 종류: IQ_EXECUTION, OQ_EXECUTION, PQ_EXECUTION. | `OQ_EXECUTION` |
| 979 | 발생 대상 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 해당 수행 테이블의 execution_id. 특정 회차·정정 결과를 가리키는 다형 참조이며 물리 FK가 아님. | `UUID` |
| 980 | 일탈 번호 | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트 내 일탈 관리 번호 | `DEV-001` |
| 981 | 일탈 제목 | `title` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시험 항목명과 일탈 번호를 이용한 보고서 표시 제목. 별도 필수 입력값이 아님. | `예상 결과 불일치` |
| 982 | 일탈 사유 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 현재 조치 개정의 일탈 사유 표시 요약. 원본 실패 사유는 source_entity_id의 수행 기록에서 조회한다. | `OQ 수행 중 예상 결과와 실제 결과 불일치` |
| 983 | 일탈 상태 | `deviation_status` | `varchar(30)` | N | N | - | Y | `'ACTION_PENDING'` | N | Y | N | Y | ACTION_PENDING(사유·조치 승인 대기), RERUN_ALLOWED(재수행 가능), COMPLETION_PENDING(완료보고 대기), CLOSED(종료). | `ACTION_PENDING` |
| 984 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 완료보고 승인 또는 사유를 입력한 종료 처리의 최종 서명 시각. | `2026-09-03T17:00:00Z` |
| 985 | 승인자 ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 완료보고 승인 또는 종료 처리의 최종 서명 사용자. | `UUID` |
| 986 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 987 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 등록 사용자 | `UUID` |
| 988 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 989 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 최종 수정 사용자 | `UUID` |
| 990 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 일탈 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 991 | 즉시 조치 | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | 현재 조치 개정의 즉시 조치 표시 요약. 회차별 승인 원문은 deviation_action_round에 보존한다. | - |
| 992 | 사유·조치 승인 상태 | `action_approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | 현재 조치 개정의 승인 상태 요약. DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED. 독립 편집하지 않는다. | `DRAFT` |
| 993 | 사유·조치 승인 워크플로우 | `action_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | current_action_round_id의 workflow_instance_id에서 동기화한 현재 승인 절차. 이전 회차는 deviation_action_round에서 조회하며 독립 편집하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 994 | 사유·조치 최종 서명 | `action_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 현재 조치 회차의 signature_id에서 동기화한 최종 승인 서명. 과거 서명은 회차별로 보존한다. | `00000000-0000-0000-0000-000000000001` |
| 995 | 최근 재수행 ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | 최근 재수행의 유효한 결과 개정 PK. 회차의 최초 재수행과 같은 논리 시험·attempt_no에 속해야 한다. source_entity_type으로 테이블을 식별한다. | `00000000-0000-0000-0000-000000000001` |
| 996 | 시정·예방 조치 | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | 화면에서 완료보고 작성 시 입력하는 시정·예방 조치. | - |
| 997 | 완료 보고 | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | 화면에서 재수행 결과를 확인한 후 입력하는 완료 보고. | - |
| 998 | 완료보고 승인 서명 | `completion_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 시정·예방 조치와 완료보고를 확인하여 승인한 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 999 | 종료 처리 사유 | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | 재수행 없이 종료할 때 화면에서 입력하는 필수 사유. | - |
| 1000 | 종료 처리 서명 | `closure_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 재수행 없이 사유를 입력하여 종료한 전자서명. 완료보고 승인과 구분. | `00000000-0000-0000-0000-000000000001` |
| 1001 | 현재 일탈 조치 개정 | `current_action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | N | - | N | Y | N | Y | 이 일탈의 마지막 조치 회차에서 최신 원문 개정. 최초 실패와 첫 조치 초안을 생성하는 트랜잭션 종료 시 필수 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deviation_1` | UNIQUE | `project_id, deviation_no` | UNIQUE(project_id, deviation_no) |
| `uq_deviation_1_rule` | 업무 검증 | `project_id, source_entity_id, deviation_no` | source_entity_id 및 rerun_execution_id는 동일 프로젝트·동일 논리 시험에 속해야 한다 |
| `rule_deviation_2` | 업무 검증 | - | RERUN_ALLOWED 전환에는 사유·조치 APPROVED와 action_signature_id가 필요하다. |
| `rule_deviation_3` | 업무 검증 | - | 일탈 원본 source_entity_id는 최초 실패 기록을 유지한다. 재수행 FAIL이면 최근 수행을 연결하고 새로운 사유·조치 승인 이력을 추가한다. |
| `rule_deviation_4` | 업무 검증 | `corrective_action, completion_report, completion_signature_id` | COMPLETION_PENDING 이후 완료보고 승인으로 종료할 때 corrective_action / completion_report / completion_signature_id 필수. 완료보고 승인 직전 최근 재수행의 유효 결과가 PASS인지 검사한다. |
| `rule_deviation_5` | 업무 검증 | `closure_reason, closure_signature_id` | 재수행 없이 종료할 때 RERUN_ALLOWED 상태, closure_reason / closure_signature_id 필수. 두 종료 경로는 구분하고 과거 서명·결과는 삭제하지 않는다. |
| `rule_deviation_6` | 업무 검증 | `approved_by, completion_signature_id, closure_signature_id` | CLOSED이면 completion_signature_id / closure_signature_id 중 정확히 하나와 approved_by / approved_at이 필수이며, 최종 서명자·서명 시각과 일치해야 한다. |
| `rule_deviation_7` | 업무 검증 | - | 재수행 판정은 rerun_execution_id의 qualification_result에서 조회한다. 원본 실패 사유·즉시 조치는 수행 기록에서 당시 값을 재현할 수 있다. |

#### 업무 규칙

종결 처리자·시각은 최종 전자서명 값과 일치시킨다.

최초 FAIL 등록과 첫 조치 회차 생성은 한 트랜잭션으로 처리한다. current_action_round_id는 이 일탈에 속하며 마지막 round_number의 is_current_revision=TRUE인 행이어야 한다. 현재 사유·조치·승인 상태·Workflow·서명은 해당 행에서 동기화하고 과거 원문은 회차 행에서 조회한다. 최근 재수행이 FAIL이면 기존 일탈을 유지하면서 다음 조치 회차를 생성한다.

완료보고·미수행 종료 서명은 deviation PK를 대상으로 하며 전자서명의 버전 규칙을 따른다. signed_payload에는 current_action_round_id, 최초 실패 PK, 최근 재수행의 정확한 결과 개정 PK, 완료보고 또는 종료 사유를 고정한다. 완료보고 종료에는 최근 재수행이 PASS이어야 한다. CLOSED 이후 사유·조치·완료보고·서명·결과 참조를 수정하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-deviation_action_round"></a>
### 62. 일탈 조치 회차 (`deviation_action_round`)

| 항목 | 정의 |
|---|---|
| 설명 | 실패 수행별 조치 원문 개정·승인과 허용된 재수행의 연결 |
| Primary Key | `action_round_id` |
| 주요 참조(FK) | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존. 승인 원문 및 과거 연결 이력은 삭제하지 않는다. |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1002 | 일탈 조치 개정 ID | `action_round_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 하나의 조치 회차에서 작성한 특정 원문 개정 식별자 | - |
| 1003 | 일탈 ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | 일탈 원본 | - |
| 1004 | 조치 회차 | `round_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 최초 실패는 1. 승인 후 재수행이 실패하면 다음 회차를 생성한다. | `1` |
| 1005 | 회차 내 원문 개정 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 실패에 대한 반려 후 수정·재상신 시 증가한다. | `1` |
| 1006 | 회차 최신 개정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 일탈·조치 회차에서 최신 작성 개정은 한 건 | - |
| 1007 | 해당 회차 실패 수행 ID | `failed_execution_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | deviation.source_entity_type에 해당하는 수행 PK. 첫 회차는 최초 실패, 다음 회차는 직전 회차의 실패한 재수행. 다형 참조이며 물리 FK 아님 | - |
| 1008 | 회차별 일탈 사유 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 해당 실패에 대한 승인 대상 사유 원문 | - |
| 1009 | 회차별 즉시 조치 | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | 해당 실패에 대해 수행한 조치 원문 | - |
| 1010 | 조치 승인 상태 | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED | - |
| 1011 | 조치 승인 절차 | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 action_round_id와 ACTION-{round_number}-{revision_number}를 대상으로 하는 DEVIATION_ACTION 결재 | - |
| 1012 | 조치 최종 승인 서명 | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 해당 조치 원문 개정의 최종 승인 서명 | - |
| 1013 | 승인으로 허용한 재수행 ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | 이 승인으로 시작한 한 번의 재수행 PK. source_entity_type으로 테이블을 식별한다. 재수행하지 않고 종료한 경우 NULL | - |
| 1014 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 원문 개정 생성 시각 | - |
| 1015 | 작성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조치 원문 작성자 | - |
| 1016 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 초안 편집 또는 관리 상태 변경 시각 | - |
| 1017 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 처리자 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deviation_action_revision` | UNIQUE | `deviation_id, round_number, revision_number` | UNIQUE (deviation_id, round_number, revision_number) |
| `uq_deviation_action_current` | UNIQUE | `deviation_id, round_number, is_current_revision` | UNIQUE (deviation_id, round_number) WHERE is_current_revision=TRUE |
| `uq_deviation_action_approved` | UNIQUE | `deviation_id, round_number, approval_status` | UNIQUE (deviation_id, round_number) WHERE approval_status='APPROVED' |
| `ck_deviation_action_sequence` | CHECK | `round_number, revision_number` | CHECK (round_number >= 1 AND revision_number >= 1) |
| `ck_deviation_action_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_deviation_action_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_action_rerun` | CHECK | `rerun_execution_id, approval_status` | CHECK (rerun_execution_id IS NULL OR approval_status='APPROVED') |
| `rule_deviation_action_chain` | 업무 검증 | `failed_execution_id, rerun_execution_id` | 실패·재수행은 일탈과 같은 프로젝트·단계·논리 시험에 속한다. 첫 실패는 일탈 source_entity_id, 다음 회차 실패는 직전 승인 회차의 재수행 또는 그 재수행과 동일 프로토콜·attempt_no에 속하는 정정 결과 개정이며 FAIL이어야 한다. 같은 회차의 개정은 failed_execution_id를 유지한다. |

#### 업무 규칙

상신 후 조치 원문을 잠근다. 반려·취소 후 내용을 바꿀 때는 같은 round_number의 새 revision_number와 PK를 생성하고 이전 행·Workflow·서명은 보존한다. 승인된 회차는 새 원문 개정을 만들지 않는다. 재수행 FAIL은 다음 round_number로 연결한다.

승인 Workflow와 서명은 deviation이 아닌 이 테이블의 정확한 action_round_id·ACTION-{round_number}-{revision_number}를 가리킨다. 사유·조치 원문과 failed_execution_id는 서명 원문에 포함한다. 승인 이후 조치 원문·실패 참조·승인 상태·최종 서명은 변경하지 않는다. 이후 최초 재수행 연결과 수정자·수정 시각 등 관리 메타데이터만 갱신할 수 있다. rerun_execution_id는 최초 연결 후 다른 회차로 덮어쓰지 않는다.

REVIEW·APPROVAL·APPROVED에는 workflow_instance_id가 필수이며 승인 Workflow의 대상·범위·버전, 최종 서명과 서명 원문은 이 조치 개정과 일치해야 한다.

재수행 생성은 일탈과 승인 회차를 잠그고 승인 상태·현재 회차·기존 재수행 부재를 검사한 뒤 시험 수행 생성, 회차 연결, 일탈 요약 갱신을 한 트랜잭션으로 처리한다. 수행 정정은 새 수행 PK를 사용하되 원래 재수행 연결을 유지하고 동일 attempt_no의 record_revision으로 정정 계보를 조회한다. 다음 실패의 판단과 완료보고는 해당 시점의 유효한 결과 개정을 명시적으로 식별한다.

[↑ 맨 위로](#top)

---

## Document

<a id="table-deliverable_document"></a>
### 63. 산출물 문서 (`deliverable_document`)

| 항목 | 정의 |
|---|---|
| 설명 | 활동별 산출물의 문서 번호와 지속 식별 |
| Primary Key | `document_id` |
| 주요 참조(FK) | `project_id, project_activity_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1018 | 문서 ID | `document_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1019 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1020 | 수행 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | project_activity.project_activity_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1021 | 문서 구분 | `document_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | STAGE_DELIVERABLE / PROTOCOL / RECORD / SUMMARY | `RECORD` |
| 1022 | 문서 번호 | `document_number` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 산출물 화면의 문서 번호. 승인 전 NULL 허용 | `OQ-R-001` |
| 1023 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1024 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1025 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1026 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_deliverable_document_1` | 업무 검증 | - | 문서의 project_activity_id는 동일 project_id에 속해야 한다. RTM은 독립 수행 활동/필수 산출물에 포함하지 않는다. |
| `rule_deliverable_document_2` | 업무 검증 | - | 문서번호가 있는 경우 프로젝트 안에서 유일하게 관리한다. 시험 프로토콜과 결과 문서는 문서 구분으로 나눈다. |

[↑ 맨 위로](#top)

---

<a id="table-deliverable_revision"></a>
### 64. 산출물 개정 (`deliverable_revision`)

| 항목 | 정의 |
|---|---|
| 설명 | 목차·본문 편집, 버전별 보기, 작성자 서명과 문서 승인 대상 |
| Primary Key | `document_revision_id` |
| 주요 참조(FK) | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1027 | 문서 개정 ID | `document_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1028 | 문서 ID | `document_id` | `uuid` | N | Y | `deliverable_document.document_id` | Y | - | N | Y | N | Y | deliverable_document.document_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1029 | 문서 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 문서 제목 | `운전 적격성 평가 결과서` |
| 1030 | 문서 표시 버전 | `version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 문서 표시 버전 | `v1.0` |
| 1031 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 개정 순번 | `1` |
| 1032 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 사유 | `시험 결과 반영` |
| 1033 | 최신 작성본 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 최신 작성본 여부 | `TRUE` |
| 1034 | 문서 승인 상태 | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED | `DRAFT` |
| 1035 | 문서 결재 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1036 | 근거 업무 승인 버전 | `source_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 화면의 승인 데이터 버전. 문서 자체 버전과 구분 | `v1.0` |
| 1037 | 근거 업무 개정 목록 | `source_refs` | `jsonb` | N | N | - | Y | - | N | N | N | Y | [{table_name,record_id,version}] 배열. 여러 업무 항목의 정확한 개정과 승인된 수행 회차를 고정 | `[{"table_name":"oq_execution","record_id":"00000000-0000-0000-0000-000000000001","version":"1-1"}]` |
| 1038 | 승인 데이터 표 | `approved_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 화면의 승인 데이터 열/행. {columns:string[],rows:string[][]} | `{"columns":["항목","결과"],"rows":[["OQ-001","Pass"]]}` |
| 1039 | Part 11 표 | `part11_table_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | QIA 산출물의 Part 11 문항·응답·판정 표. 다른 활동은 NULL | - |
| 1040 | 개정 이력 표 | `revision_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 작성 당시 개정 이력 열/행 | `{"columns":["버전","사유"],"rows":[["v1.0","최초 작성"]]}` |
| 1041 | 재승인 원인 활동 | `reapproval_source_stage` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 상위 활동 변경으로 재승인이 필요한 경우의 활동 코드. RTM 제외 | `URS` |
| 1042 | 재승인 사유 | `reapproval_reason` | `text` | N | N | - | N | - | N | N | N | Y | 재승인 사유 | `URS 개정으로 내용 재검토` |
| 1043 | 승인 유효성 상실 시각 | `invalidated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 승인 유효성 상실 시각 | `2026-09-01T00:00:00Z` |
| 1044 | 최종 승인자 ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1045 | 최종 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 승인 시각 | `2026-09-01T00:00:00Z` |
| 1046 | PDF 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 생성 PDF를 보관한 경우 연결 | `00000000-0000-0000-0000-000000000001` |
| 1047 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1048 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1049 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1050 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1051 | 문서 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 문서 상신 시 고정한 동일 프로젝트의 기준. 상신·승인 시 필수. 이전 수행은 각 수행의 기준을 함께 표시한다. | - |
| 1052 | 평가 질문·판정 규칙 원문 | `evaluation_definition_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | QIA/FRA 문서의 근거 평가별 질문·규칙 버전과 실제 정의, 입력·결과를 고정한 배열. 평가와 무관한 문서는 NULL | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deliverable_revision_1` | UNIQUE | `document_id, revision_number` | UNIQUE(document_id,revision_number) |
| `uq_deliverable_revision_1_2` | UNIQUE | `document_id, version` | UNIQUE(document_id,version) |
| `uq_deliverable_revision_1_rule` | 업무 검증 | `document_id, version, revision_number, is_current_version` | 문서별 is_current_version=TRUE는 최대 1개 |
| `rule_deliverable_revision_2` | 업무 검증 | - | 문서 승인과 근거 업무 항목 승인은 별도이다. source_refs는 여러 원본 항목의 정확한 개정·프로젝트·승인 여부를 검증하고 당시 표와 함께 보존한다. |
| `rule_deliverable_revision_3` | 업무 검증 | - | APPROVED이면 문서 결재 이력·최종 승인자·시각이 필요하다. 승인된 본문/표/근거 연결은 변경하지 않고 새 개정을 작성한다. |
| `rule_deliverable_revision_4` | 업무 검증 | - | 이후 원본이 바뀌어도 과거 승인본을 삭제하지 않는다. 현재 유효성은 invalidated_at과 재승인 사유로 별도 판단한다. |
| `uq_document_current_revision` | UNIQUE | `document_id` | UNIQUE (document_id) WHERE is_current_version=TRUE |

#### 업무 규칙

PDF 생성 상태는 문서 승인 상태를 대신하지 않는다. VP 원본 목차는 vp_section이고, VP 산출물은 승인된 원본을 반영하므로 같은 내용을 두 곳에서 독립 편집하지 않는다.

IQ/OQ/PQ의 source_refs에는 해당 평가 개정과 문서에 포함된 시험 항목·수행의 정확한 PK·버전을 모두 기록한다. 프로토콜 문서는 평가 item_revision_refs의 전체 구성을 고정하며 결과 문서는 그 구성에 속한 수행의 정확한 회차·결과 개정을 연결한다. 문서만 개정할 때 같은 평가·시험 참조를 그대로 재사용할 수 있다. system_baseline_id와 참조 대상은 같은 프로젝트여야 한다. 다른 기준에서 수행한 결과를 포함하면 해당 수행의 기준과 재사용 판단 근거를 본문에 명시한다.

문서 상신·승인 시 system_baseline_id는 필수이며, QIA/FRA 문서는 evaluation_definition_snapshot도 필수이다. 해당 사본은 입력과 버전별 실제 정의로 결과를 재계산할 수 있어야 하며 단순 결과 표만 저장하지 않는다. 상신 후 본문·구성·질문 및 규칙 사본은 잠그고 반려 후 수정은 새 문서 개정으로 처리한다. 생성 PDF는 파일 생성 이후 연결하되 이미 서명한 업무 원문을 바꾸지 않으며, PDF 파일의 원본 동일성은 file_asset의 해시로 별도 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-deliverable_section"></a>
### 65. 산출물 목차/본문 (`deliverable_section`)

| 항목 | 정의 |
|---|---|
| 설명 | 문서 편집 화면에서 추가·수정·삭제·순서 변경하는 섹션 |
| Primary Key | `section_id` |
| 주요 참조(FK) | `document_revision_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1053 | 섹션 ID | `section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1054 | 문서 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | Y | - | N | Y | N | Y | deliverable_revision.document_revision_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1055 | 섹션 키 | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 섹션 키 | `purpose` |
| 1056 | 목차 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 목차 제목 | `목적` |
| 1057 | 본문 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | 본문 | `본 시험의 목적을 기술한다.` |
| 1058 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1059 | 작성 출처 | `source_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MANUAL / AI / APPROVED / HISTORY / PART11 | `MANUAL` |
| 1060 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1061 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1062 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1063 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deliverable_section_1` | UNIQUE | `document_revision_id, section_key` | UNIQUE(document_revision_id,section_key) |
| `uq_deliverable_section_1_2` | UNIQUE | `document_revision_id, sort_order` | UNIQUE(document_revision_id,sort_order) |
| `ck_deliverable_section_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_deliverable_section_2` | 업무 검증 | - | MANUAL/AI 본문은 승인 전 편집 가능하다. APPROVED/HISTORY/PART11 섹션은 해당 스냅샷에서 표시한다. 승인된 문서 섹션은 직접 수정·삭제하지 않는다. |

[↑ 맨 위로](#top)

---

## Report

<a id="table-report_generation"></a>
### 66. 리포트 생성 작업 (`report_generation`)

| 항목 | 정의 |
|---|---|
| 설명 | 화면에서 요청한 산출물·감사 PDF 생성과 다운로드 결과 |
| Primary Key | `report_generation_id` |
| 주요 참조(FK) | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 리포트 생성 결과 및 실행이력을 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1064 | 리포트 생성 ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 리포트 생성 작업 고유 식별자 | `UUID` |
| 1065 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 단위 리포트인 경우 연결하며 전체 시스템 또는 조직 단위 리포트는 NULL 허용 | `UUID` |
| 1066 | 리포트 유형 | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DELIVERABLE / AUDIT_TRAIL. 화면의 산출물 또는 감사 내보내기 | `AUDIT_TRAIL` |
| 1067 | 리포트명 | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 사용자에게 표시되는 생성 리포트명 | `2026년 8월 Audit Trail 리포트` |
| 1068 | 조회 시작일시 | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 시작일시. 조회기간이 없는 리포트는 NULL 허용 | `2026-08-01T00:00:00Z` |
| 1069 | 조회 종료일시 | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 종료일시. period_from보다 빠를 수 없음 | `2026-08-31T23:59:59Z` |
| 1070 | 조회 조건 | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 화면에서 선택한 기간·사용자·역할·메뉴 등 내보내기 필터 | `{"action_types":["CREATE","UPDATE"]}` |
| 1071 | 출력 형식 | `output_format` | `varchar(20)` | N | N | - | Y | `'PDF'` | N | Y | N | Y | 출력 형식 PDF | `PDF` |
| 1072 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1073 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PDF/내보내기를 요청한 사용자 | `UUID` |
| 1074 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 사용자가 내보내기를 요청한 시각 | `2026-09-02T15:00:00Z` |
| 1075 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 생성이 시작된 시각 | `2026-09-02T15:00:05Z` |
| 1076 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 생성 성공 또는 실패가 확정된 시각 | `2026-09-02T15:01:30Z` |
| 1077 | 결과 파일 ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 결과를 파일 자산으로 보관한 경우 연결. 다운로드만 수행하면 NULL 가능 | `UUID` |
| 1078 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인을 분류하는 시스템 오류 코드 | `REPORT_FILE_CREATE_FAILED` |
| 1079 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 리포트 생성 실패 상세 내용. 비밀번호, 토큰 등 민감정보는 저장하지 않음 | `결과 파일 저장 중 오류 발생` |
| 1080 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 레코드 생성 시각(UTC) | `2026-09-02T15:00:00Z` |
| 1081 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 화면에서 내보내기를 요청한 사용자 | `UUID` |
| 1082 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 최종 수정 시각(UTC) | `2026-09-02T15:01:30Z` |
| 1083 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 내보내기 처리 결과의 수정 사용자. 자동 상태 반영이면 NULL 가능 | `UUID` |
| 1084 | 산출물 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | deliverable_revision.document_revision_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_report_generation_1` | 업무 검증 | `document_revision_id` | DELIVERABLE이면 document_revision_id 필수이며 해당 프로젝트와 일치해야 한다. AUDIT_TRAIL이면 기간/필터로 조회 범위를 고정한다. |
| `rule_report_generation_2` | 업무 검증 | `period_from, period_to` | period_from과 period_to가 모두 있으면 period_from<=period_to. generation_status에는 자동 재시도 대기 상태를 두지 않는다. |

#### 업무 규칙

본문·목차·버전·문서 승인 상태는 deliverable_revision과 deliverable_section에 저장한다. 이 테이블은 내보내기 결과만 관리한다.

[↑ 맨 위로](#top)

---

## AI

<a id="table-ai_generation_job"></a>
### 67. AI 생성 작업 (`ai_generation_job`)

| 항목 | 정의 |
|---|---|
| 설명 | 화면의 AI 초안 생성 요청, 입력 조건 및 진행 결과 |
| Primary Key | `ai_job_id` |
| 주요 참조(FK) | `project_id, requested_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1085 | AI 작업 ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 생성 작업 식별자 | `UUID` |
| 1086 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 연결 | `UUID` |
| 1087 | 작업 유형 | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 1088 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_REVISION | `IQ_ITEM` |
| 1089 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 기존 대상이면 해당 개정 PK. 새 항목을 생성하는 미리보기는 NULL 허용 | `UUID` |
| 1090 | AI 모델명 | `model_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 생성에 사용한 모델 식별자. 확인한 경우 기록하며 미확인 시 NULL을 허용한다. | - |
| 1091 | 입력 파라미터 | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | 화면 입력 목적·범위·기준 URS·생성 개수 또는 생성할 문서 목차 | `{"purpose":"접근권한 검증","scope":"로그인","count":3}` |
| 1092 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1093 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 요청 사용자 | `UUID` |
| 1094 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 요청 접수 시각 | `2026-09-02T00:00:00Z` |
| 1095 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 시작 시각 | `2026-09-02T00:00:00Z` |
| 1096 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 완료 시각 | `2026-09-02T00:00:00Z` |
| 1097 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인 코드 | `LLM_TIMEOUT` |
| 1098 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 실패 상세 메시지 | `모델 응답시간 초과` |
| 1099 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1100 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 1101 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1102 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_ai_generation_job_1` | 업무 검증 | `target_entity_id` | ITEM_GENERATION의 신규 항목은 적용 전 target_entity_id NULL을 허용한다. DOCUMENT_GENERATION은 해당 문서 개정에 연결한다. |
| `rule_ai_generation_job_2` | 업무 검증 | - | 생성 완료가 업무 항목 승인이나 문서 승인을 의미하지 않는다. 선택·적용 후 일반 편집/승인 절차를 따른다. |

[↑ 맨 위로](#top)

---

<a id="table-ai_generation_result"></a>
### 68. AI 생성 결과 (`ai_generation_result`)

| 항목 | 정의 |
|---|---|
| 설명 | AI 생성 결과 집합 및 채택 상태 관리 |
| Primary Key | `ai_result_id` |
| 주요 참조(FK) | `ai_job_id, selected_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1103 | AI 결과 ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 결과 식별자 | `UUID` |
| 1104 | AI 작업 ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | 상위 AI 작업 | `UUID` |
| 1105 | 결과 제목 | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | 결과 제목 | `OQ 테스트 초안` |
| 1106 | 선택 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 사용자 채택 여부 | `TRUE` |
| 1107 | 반영 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 실제 산출물 반영 여부 | `TRUE` |
| 1108 | 선택 사용자 ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 채택 사용자 | `UUID` |
| 1109 | 선택 시각 | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 채택 시각 | `2026-09-02T00:00:00Z` |
| 1110 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1111 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 1112 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1113 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |

#### 업무 규칙

AI 결과 묶음이다. 항목별 선택·적용은 ai_result_item을 기준으로 하고 is_selected/is_applied는 해당 묶음의 요약이다.

[↑ 맨 위로](#top)

---

<a id="table-ai_result_item"></a>
### 69. AI 생성 결과 항목 (`ai_result_item`)

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
| 1114 | AI 결과 항목 ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 결과 항목 식별자 | `UUID` |
| 1115 | AI 결과 ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | 상위 결과 참조 | `UUID` |
| 1116 | 항목 순번 | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 결과 표시 순서 | `1` |
| 1117 | 항목 유형 | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 1118 | 제목 | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 생성 항목 제목 | `전자서명 기록` |
| 1119 | 본문 내용 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | AI가 생성한 시험/요구사항/위험평가/목차별 초안 내용 | `내용` |
| 1120 | 적용 대상 유형 | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 적용한 업무 테이블 유형: REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_SECTION | `IQ_ITEM` |
| 1121 | 적용 대상 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 실제로 적용하여 생성/수정한 개정 PK. 적용 전 NULL | `UUID` |
| 1122 | 채택 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 사용자 채택 여부 | `TRUE` |
| 1123 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1124 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 생성 사용자 | `UUID` |
| 1125 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1126 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 수정 사용자 | `UUID` |
| 1127 | 적용 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 적용 여부 | `FALSE` |
| 1128 | 적용 시각 | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 적용 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_ai_result_item_1` | UNIQUE | `ai_result_id, item_order` | UNIQUE(ai_result_id,item_order) |
| `uq_ai_result_item_1_rule` | 업무 검증 | `ai_result_id, item_order, is_applied` | is_applied=TRUE이면 실제 적용 대상 유형·PK와 applied_at이 필요하다 |
| `rule_ai_result_item_2` | 업무 검증 | - | 문서 목차별 결과는 DELIVERABLE_SECTION으로 연결한다. 승인된 항목/문서를 덮어쓰지 않으며 수정은 초안 또는 새 개정에 적용한다. |

[↑ 맨 위로](#top)

---

## Regulation

<a id="table-regulatory_source"></a>
### 70. 규정 근거 문서 (`regulatory_source`)

| 항목 | 정의 |
|---|---|
| 설명 | URS·FRA·라이브러리에서 참조하는 규정·지침·SOP의 문서판 |
| Primary Key | `regulatory_source_id` |
| 주요 참조(FK) | `file_id, verified_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1129 | 규정 문서 ID | `regulatory_source_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1130 | 문서 코드 | `source_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 문서 코드 | `REF-001` |
| 1131 | 문서 유형 | `source_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | REGULATION / GUIDELINE / INTERNAL_SOP | `GUIDELINE` |
| 1132 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 문서명 | `검토용 기준 문서` |
| 1133 | 판본/개정 | `edition` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 판본/개정 | `Rev.1` |
| 1134 | 발행기관/부서 | `issuing_body` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 발행기관/부서 | `품질보증팀` |
| 1135 | 출처 URL | `source_url` | `text` | N | N | - | N | - | N | N | N | Y | 출처 URL | `https://example.com/reference` |
| 1136 | 원본 첨부 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | file_asset.file_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1137 | 적용 시작일 | `effective_from` | `date` | N | N | - | N | - | N | N | N | Y | 적용 시작일 | `2026-09-01` |
| 1138 | 적용 종료일 | `effective_to` | `date` | N | N | - | N | - | N | N | N | Y | 적용 종료일 | `2026-09-01` |
| 1139 | 검토 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / VERIFIED / RETIRED | `DRAFT` |
| 1140 | 확인자 ID | `verified_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1141 | 확인 시각 | `verified_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 확인 시각 | `2026-09-01T00:00:00Z` |
| 1142 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1143 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1144 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1145 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_regulatory_source_1` | UNIQUE | `source_code, edition` | UNIQUE(source_code,edition) |
| `uq_regulatory_source_1_rule` | 업무 검증 | `source_code, edition` | 다른 언어판은 별도 source_code로 식별한다 |
| `rule_regulatory_source_2` | 업무 검증 | - | VERIFIED이면 확인자·시각 및 URL/원본 첨부 중 하나가 필요하다. 적용 종료일은 시작일 이전일 수 없다. |
| `rule_regulatory_source_3` | 업무 검증 | - | 새 판본은 새 행으로 등록한다. 승인된 업무가 참조한 문서판은 삭제하거나 새 내용으로 덮어쓰지 않는다. |

#### 업무 규칙

규정 근거는 재사용 기준 데이터로 관리한다. 수기 근거는 문서판과 조항을 확인한 후 연결하며, 검증 완료 상태는 확인자와 확인 시각을 기록한 경우에만 부여한다.

[↑ 맨 위로](#top)

---

<a id="table-regulatory_clause"></a>
### 71. 규정 조항 (`regulatory_clause`)

| 항목 | 정의 |
|---|---|
| 설명 | 규정 근거 문서판에 속한 조항과 표시 문구 |
| Primary Key | `regulatory_clause_id` |
| 주요 참조(FK) | `regulatory_source_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1146 | 조항 ID | `regulatory_clause_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1147 | 규정 문서 ID | `regulatory_source_id` | `uuid` | N | Y | `regulatory_source.regulatory_source_id` | Y | - | N | Y | N | Y | regulatory_source.regulatory_source_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1148 | 조항 코드 | `clause_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 조항 코드 | `4.1` |
| 1149 | 조항 제목 | `title` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | 조항 제목 | `접근 관리` |
| 1150 | 조항 요약 | `summary` | `text` | N | N | - | Y | - | N | N | N | Y | 조항 요약 | `해당 업무에 적용할 요구사항 요약` |
| 1151 | 원문 위치 | `source_locator` | `text` | N | N | - | N | - | N | N | N | Y | 원문 위치 | `제4장 1절` |
| 1152 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1153 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 사용 여부 | `TRUE` |
| 1154 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1155 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1156 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1157 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_regulatory_clause_1` | UNIQUE | `regulatory_source_id, clause_code` | UNIQUE(regulatory_source_id,clause_code) |
| `ck_regulatory_clause_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_regulatory_clause_2` | 업무 검증 | - | 신규 선택은 VERIFIED 문서판의 활성 조항을 사용한다. 이미 승인에 사용한 조항이 비활성화되어도 과거 참조를 유지한다. |

[↑ 맨 위로](#top)

---

<a id="table-requirement_regulation"></a>
### 72. 요구사항 규정 근거 (`requirement_regulation`)

| 항목 | 정의 |
|---|---|
| 설명 | 업무 항목과 적용 규정 조항의 복수 연결 |
| Primary Key | `requirement_regulation_id` |
| 주요 참조(FK) | `requirement_id, regulatory_clause_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1158 | 근거 연결 ID | `requirement_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1159 | 업무 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | requirement.requirement_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1160 | 규정 조항 ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | regulatory_clause.regulatory_clause_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1161 | 적용 근거 | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | 적용 근거 | `접근권한 요구사항에 적용` |
| 1162 | 인용 문구 | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | 적용 당시 문서명·판본·조항 표시를 보존 | `검토용 기준 문서 Rev.1 / 4.1` |
| 1163 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1164 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1165 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1166 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1167 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_requirement_regulation_1` | UNIQUE | `requirement_id, regulatory_clause_id` | UNIQUE(requirement_id,regulatory_clause_id) |
| `ck_requirement_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_requirement_regulation_2` | 업무 검증 | - | 업무 승인 시 당시 조항과 인용 문구를 보존한다. 라이브러리에서 URS로 적용할 때 근거 관계도 복사하며 이후 템플릿 변경을 기존 URS에 자동 전파하지 않는다. |

[↑ 맨 위로](#top)

---

<a id="table-library_item_regulation"></a>
### 73. 라이브러리 규정 근거 (`library_item_regulation`)

| 항목 | 정의 |
|---|---|
| 설명 | 업무 항목과 적용 규정 조항의 복수 연결 |
| Primary Key | `library_item_regulation_id` |
| 주요 참조(FK) | `library_id, regulatory_clause_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1168 | 근거 연결 ID | `library_item_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1169 | 업무 항목 ID | `library_id` | `uuid` | N | Y | `library_item.library_id` | Y | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1170 | 규정 조항 ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | regulatory_clause.regulatory_clause_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1171 | 적용 근거 | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | 적용 근거 | `접근권한 요구사항에 적용` |
| 1172 | 인용 문구 | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | 적용 당시 문서명·판본·조항 표시를 보존 | `검토용 기준 문서 Rev.1 / 4.1` |
| 1173 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1174 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1175 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1176 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1177 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_library_item_regulation_1` | UNIQUE | `library_id, regulatory_clause_id` | UNIQUE(library_id,regulatory_clause_id) |
| `ck_library_item_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_library_item_regulation_2` | 업무 검증 | - | 업무 승인 시 당시 조항과 인용 문구를 보존한다. 라이브러리에서 URS로 적용할 때 근거 관계도 복사하며 이후 템플릿 변경을 기존 URS에 자동 전파하지 않는다. |

[↑ 맨 위로](#top)

---
