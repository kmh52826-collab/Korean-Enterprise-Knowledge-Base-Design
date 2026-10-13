<a id="top"></a>

# 데이터 테이블 정의서

> Validation Management Platform 데이터 모델 문서  
> **예시값 기준**: 개인정보 및 민감정보는 비식별 샘플 값으로 표기

업무 영역별 테이블 연결은 **[ERD 명세서](./erd-specifications.md)**, 영역 간 전체 구조는 **[전체 데이터 구조도](./erd-overview.md)** 를 참고하세요.

## 목차

- [1. 문서 개요](#1-문서-개요)
- [2. 표기 기준](#2-표기-기준)
- [3. 테이블 목록](#3-테이블-목록)
- [4. 도메인별 바로가기](#4-도메인별-바로가기)
- [5. 상세 테이블 및 컬럼 정의](#5-상세-테이블-및-컬럼-정의)

## 1. 문서 개요

- 전체 테이블: **76개**
- 전체 컬럼: **1,253개**
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

민감=Y인 값은 허용된 사용자·처리 목적에 한해 조회·내보내기한다. 실명, 이메일, IP, 감사자·승인자 성명 및 개인정보를 포함할 수 있는 Audit JSON을 이 분류에 포함한다. password_hash·previous_password_hash와 user_signature_credential.signature_password_hash·previous_signature_password_hash는 응답·일반 내보내기·Audit JSON에서 제외하고 Audit=N을 유지한다.

NN=N은 승인·완료 시에도 항상 선택이라는 뜻은 아닙니다. 조건부 필수값은 해당 테이블의 제약조건·업무 규칙을 따릅니다. 복수 테이블을 가리키는 유형+ID 및 JSON 내부 참조는 실제 FK와 구분합니다.

DB의 PK·FK·CHECK·UNIQUE는 저장 시 강제하는 제약이다. 제약조건 표는 한 행에 하나의 DB 제약 또는 업무 검증을 정의한다. 조건부 UNIQUE는 해당 WHERE 조건의 행에만 적용한다. 업무 검증은 서버에서 수행하고 UI의 선택 제한만으로 대체하지 않는다.

참조 저장·상신·승인 시 대상 존재, 부모·프로젝트·프로토콜 개정의 일치 및 현재 사용·승인 가능 상태를 검사한다. 다형 참조와 JSON 내부 참조는 허용된 실제 테이블·PK·버전과 객체 형식을 검사한다. 프로젝트 업무끼리의 연결은 소속 프로젝트 일치를 요구하고 공용 규정·라이브러리 등 프로젝트 외 자료는 해당 자료의 조회 및 사용 권한을 검사한다. 신규 업무 연결에는 삭제된 대상과 다른 프로젝트의 업무 기록을 허용하지 않는다. 과거 승인 참조의 조회는 당시 원문을 유지하고 이후의 폐기·유효성 상실 상태와 구분한다.

동시 요청은 서버 트랜잭션과 대상 행 잠금 또는 저장 버전 비교로 제어한다. 중복 승인·번호 발급·최신 개정 전환·진행 중 종료 요청 생성은 검사와 저장을 분리하지 않는다. 저장 버전이 바뀐 요청은 충돌로 처리하고 이전 화면의 값으로 최신 데이터를 덮어쓰지 않는다. 상태 전환·서명·감사기록은 함께 확정하거나 함께 취소하며 감사 변경 필드를 임의로 생략하지 않는다.

버전별 질문·판정 규칙 정의(QIA Part 11·GxP 질문, FRA 위험 판정, 활동 선후행 판정 의미)는 별도 마스터 테이블에 두지 않고 버전 이름으로 식별하는 서버 배포 설정으로 관리한다. 같은 버전 이름에 다른 정의를 배포하지 않으며, 각 평가·문서·프로젝트에 복사한 원문(question_set_snapshot, rule_snapshot, evaluation_definition_snapshot, dependency_snapshot)을 감사 근거로 삼는다.

승인 상태를 가진 업무 개정은 상신(REVIEW) 이후 본문·하위 절차·구성·첨부를 잠근다. 상신 전 DRAFT의 저장은 같은 행을 갱신하고 변경 전후 값을 Audit Trail에 기록한다. 반려(REJECTED)와 취소(CANCELLED)는 해당 개정의 종결 상태이며 DRAFT로 되돌리지 않는다. 반려·취소 후의 수정과 승인 후의 변경은 같은 논리키의 새 개정(DRAFT)으로 처리하고, 종결된 개정과 그 Workflow·전자서명은 보존한다(인벤토리는 revision_number 증가, 시험 수행 결과는 같은 attempt_no의 record_revision 증가). 화면에서는 이를 '수정'으로 표시할 수 있으나 DB에는 새 개정 행이 생성된다.

## 3. 테이블 목록

| No | Domain | 논리 테이블명 | 물리 테이블명 | PK | 주요 참조(FK) | GxP | Audit |
|---:|---|---|---|---|---|:---:|:---:|
| 1 | Organization | 조직/고객사 | [`organization`](#table-organization) | `organization_id` | - | High | Y |
| 2 | Security | 사용자 | [`app_user`](#table-app_user) | `user_id` | `organization_id` | High | Y |
| 3 | Security | 역할 | [`role`](#table-role) | `role_id` | - | High | Y |
| 4 | Security | 사용자 역할 | [`user_role`](#table-user_role) | `user_role_id` | `user_id, role_id, created_by, updated_by` | High | Y |
| 5 | Security | 사용자 그룹 | [`user_group`](#table-user_group) | `user_group_id` | `organization_id, created_by, updated_by` | High | Y |
| 6 | Security | 사용자 그룹 구성원 | [`user_group_member`](#table-user_group_member) | `user_group_member_id` | `user_group_id, user_id, created_by, updated_by` | High | Y |
| 7 | Security | 사용자 그룹 역할 | [`group_role`](#table-group_role) | `group_role_id` | `user_group_id, role_id, project_id, created_by, updated_by` | High | Y |
| 8 | Security | 메뉴·프로젝트 접근권한 | [`access_permission_grant`](#table-access_permission_grant) | `grant_id` | `user_id, user_group_id, project_id, created_by, updated_by` | High | Y |
| 9 | Security | 인벤토리 업무 역할 | [`inventory_role_grant`](#table-inventory_role_grant) | `grant_id` | `user_id, user_group_id, created_by, updated_by` | High | Y |
| 10 | Security | 전자서명 비밀번호 | [`user_signature_credential`](#table-user_signature_credential) | `credential_id` | `user_id, reset_requested_by, created_by, updated_by` | High | Y |
| 11 | Compliance | 전자서명 | [`electronic_signature`](#table-electronic_signature) | `signature_id` | `signer_id` | Critical | Y |
| 12 | Compliance | Audit Trail | [`audit_trail`](#table-audit_trail) | `audit_id` | `actor_id` | Critical | Y |
| 13 | File | 파일 자산 | [`file_asset`](#table-file_asset) | `file_id` | `uploader_id` | High | Y |
| 14 | File | 파일 악성코드 검사 작업 | [`file_scan_job`](#table-file_scan_job) | `scan_job_id` | `previous_scan_job_id, project_id, uploaded_by, requested_by, result_file_id` | High | Y |
| 15 | File | 증적 파일 연결 | [`evidence_link`](#table-evidence_link) | `evidence_link_id` | `project_id, file_id, created_by, updated_by` | High | Y |
| 16 | File | 파일 연결 예약 | [`file_link_request`](#table-file_link_request) | `link_request_id` | `project_id, file_id, evidence_link_id, created_by, updated_by` | High | Y |
| 17 | System | 시스템/장비 식별 정보 | [`system_asset`](#table-system_asset) | `system_id` | `organization_id` | High | Y |
| 18 | System | 시스템 인벤토리 개정 이력 | [`system_asset_revision`](#table-system_asset_revision) | `system_revision_id` | `system_id, processed_by, signature_id` | High | Y |
| 19 | Library | 라이브러리 항목 마스터 | [`library_item`](#table-library_item) | `library_id` | - | High | Y |
| 20 | Validation | Validation 프로젝트 | [`validation_project`](#table-validation_project) | `project_id` | `system_id, created_by, updated_by, closure_requested_by, current_closure_request_id, current_system_baseline_id` | High | Y |
| 21 | Validation | 프로젝트 검증 대상 개정 | [`project_system_baseline`](#table-project_system_baseline) | `baseline_id` | `project_id, system_revision_id, adopted_by` | Critical | Y |
| 22 | Validation | 프로젝트 참여자 | [`project_member`](#table-project_member) | `project_member_id` | `project_id, user_id, role_id, created_by, updated_by` | High | Y |
| 23 | Validation | 밸리데이션 활동 마스터 | [`validation_activity`](#table-validation_activity) | `activity_id` | `created_by, updated_by` | High | Y |
| 24 | Validation | 프로젝트 수행 활동 | [`project_activity`](#table-project_activity) | `project_activity_id` | `project_id, activity_id, created_by, updated_by` | Critical | Y |
| 25 | Validation | 활동 선후행 조건 | [`activity_dependency`](#table-activity_dependency) | `activity_dependency_id` | `successor_activity_id, predecessor_activity_id, created_by, updated_by` | Critical | Y |
| 26 | Validation | 프로젝트 종료 요청 | [`project_closure_request`](#table-project_closure_request) | `closure_request_id` | `project_id, requested_by, workflow_instance_id, created_by, updated_by` | High | Y |
| 27 | VP | 검증 계획 | [`vp_plan`](#table-vp_plan) | `vp_id` | `project_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 28 | VP | 검증 계획 목차 | [`vp_section`](#table-vp_section) | `vp_section_id` | `vp_id, created_by, updated_by` | High | Y |
| 29 | QIA | 품질 영향 평가 헤더 | [`qia_assessment`](#table-qia_assessment) | `qia_id` | `project_id, created_by, updated_by` | High | Y |
| 30 | QIA | QIA 모듈 상세 평가 | [`qia_module_item`](#table-qia_module_item) | `qia_module_item_id` | `qia_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 31 | QIA | QIA 프로세스 평가 | [`qia_process`](#table-qia_process) | `qia_process_id` | `qia_module_item_id, created_by, updated_by` | High | Y |
| 32 | VA | 공급업체 감사 평가 | [`vendor_audit`](#table-vendor_audit) | `audit_id` | `project_id, auditor_user_id, file_id, workflow_instance_id, created_by, updated_by, disposal_workflow_id` | High | Y |
| 33 | URS | 사용자 요구사항 명세 | [`requirement`](#table-requirement) | `requirement_id` | `project_id, created_by, workflow_instance_id, updated_by, disposal_workflow_id, source_library_id` | High | Y |
| 34 | FDS | 기능 설계 명세서 | [`fds_spec`](#table-fds_spec) | `fds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 35 | DDS | 상세 설계 명세서 | [`dds_spec`](#table-dds_spec) | `dds_id` | `project_id, created_by, updated_by, workflow_instance_id, disposal_workflow_id, file_id, uploaded_by` | High | Y |
| 36 | DQ | 설계 적격성 평가 | [`dq_assessment`](#table-dq_assessment) | `dq_id` | `project_id, created_by, updated_by` | High | Y |
| 37 | DQ | DQ 상세 평가 항목 | [`dq_item`](#table-dq_item) | `dq_item_id` | `dq_id, requirement_id, created_by, updated_by, fds_revision_id, dds_revision_id, executed_by, workflow_instance_id, disposal_workflow_id` | High | Y |
| 38 | FRA | 기능 위험평가 | [`fra_assessment`](#table-fra_assessment) | `fra_id` | `project_id, created_by, updated_by` | High | Y |
| 39 | FRA | FRA 위험 상세 항목 | [`fra_item`](#table-fra_item) | `fra_item_id` | `fra_id, requirement_id, created_by, updated_by, sop_clause_id, workflow_instance_id, disposal_workflow_id, source_library_id` | High | Y |
| 40 | IQ | 설치 적격성 평가 | [`iq_assessment`](#table-iq_assessment) | `iq_id` | `project_id, created_by, updated_by` | High | Y |
| 41 | IQ | IQ 상세 테스트 항목 | [`iq_item`](#table-iq_item) | `iq_item_id` | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 42 | IQ | IQ 시험 절차 | [`iq_step`](#table-iq_step) | `step_id` | `iq_item_id, created_by, updated_by` | High | Y |
| 43 | IQ | IQ 시험 수행 | [`iq_execution`](#table-iq_execution) | `execution_id` | `iq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 44 | IQ | IQ 절차 수행 | [`iq_step_execution`](#table-iq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 45 | OQ | 운전 적격성 평가 | [`oq_assessment`](#table-oq_assessment) | `oq_id` | `project_id, created_by, updated_by` | High | Y |
| 46 | OQ | OQ 상세 테스트 항목 | [`oq_item`](#table-oq_item) | `oq_item_id` | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 47 | OQ | OQ 시험 절차 | [`oq_step`](#table-oq_step) | `step_id` | `oq_item_id, created_by, updated_by` | High | Y |
| 48 | OQ | OQ 시험 수행 | [`oq_execution`](#table-oq_execution) | `execution_id` | `oq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 49 | OQ | OQ 절차 수행 | [`oq_step_execution`](#table-oq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 50 | PQ | 성능 적격성 평가 | [`pq_assessment`](#table-pq_assessment) | `pq_id` | `project_id, created_by, updated_by` | High | Y |
| 51 | PQ | PQ 상세 테스트 항목 | [`pq_item`](#table-pq_item) | `pq_item_id` | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` | High | Y |
| 52 | PQ | PQ 시험 절차 | [`pq_step`](#table-pq_step) | `step_id` | `pq_item_id, created_by, updated_by` | High | Y |
| 53 | PQ | PQ 시험 수행 | [`pq_execution`](#table-pq_execution) | `execution_id` | `pq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` | High | Y |
| 54 | PQ | PQ 절차 수행 | [`pq_step_execution`](#table-pq_step_execution) | `step_execution_id` | `execution_id, step_id, confirmed_by, created_by, updated_by` | High | Y |
| 55 | VSR | 밸리데이션 종합 보고서 | [`vsr_assessment`](#table-vsr_assessment) | `vsr_id` | `project_id, created_by, updated_by, workflow_instance_id, approval_signature_id` | Critical | Y |
| 56 | VSR | VSR 활동 요약 항목 | [`vsr_item`](#table-vsr_item) | `vsr_item_id` | `vsr_id, created_by, updated_by, project_activity_id, document_revision_id` | Critical | Y |
| 57 | Workflow | Workflow 인스턴스 | [`workflow_instance`](#table-workflow_instance) | `workflow_instance_id` | `requested_by, created_by, updated_by, project_id, workflow_config_id` | Critical | Y |
| 58 | Workflow | Workflow 단계 | [`workflow_step`](#table-workflow_step) | `workflow_step_id` | `workflow_instance_id, created_by, updated_by` | Critical | Y |
| 59 | Workflow | 결재 단계 담당자 | [`workflow_step_assignee`](#table-workflow_step_assignee) | `assignment_id` | `workflow_step_id, assignee_id, substitute_user_id, created_by, updated_by` | Critical | Y |
| 60 | Workflow | 승인 처리 이력 | [`approval_action`](#table-approval_action) | `approval_action_id` | `workflow_step_id, workflow_step_assignee_id, actor_id, signature_id` | Critical | Y |
| 61 | Workflow | 프로젝트 결재선 설정 | [`project_workflow_config`](#table-project_workflow_config) | `config_id` | `project_id, project_activity_id, applied_signature_id, created_by, updated_by` | Critical | Y |
| 62 | Traceability | 공통 추적 관계 | [`traceability_link`](#table-traceability_link) | `traceability_link_id` | `project_id, created_by, updated_by` | Critical | Y |
| 63 | Deviation | 일탈 관리 | [`deviation`](#table-deviation) | `deviation_id` | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id, current_completion_revision_id` | Critical | Y |
| 64 | Deviation | 일탈 조치 회차 | [`deviation_action_round`](#table-deviation_action_round) | `action_round_id` | `deviation_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 65 | Deviation | 일탈 종결 보고 개정 | [`deviation_completion_revision`](#table-deviation_completion_revision) | `completion_revision_id` | `deviation_id, action_round_id, workflow_instance_id, signature_id, created_by, updated_by` | Critical | Y |
| 66 | Document | 산출물 문서 | [`deliverable_document`](#table-deliverable_document) | `document_id` | `project_id, project_activity_id, created_by, updated_by` | Critical | Y |
| 67 | Document | 산출물 개정 | [`deliverable_revision`](#table-deliverable_revision) | `document_revision_id` | `document_id, workflow_instance_id, approved_by, file_id, created_by, updated_by, system_baseline_id` | Critical | Y |
| 68 | Document | 산출물 목차/본문 | [`deliverable_section`](#table-deliverable_section) | `section_id` | `document_revision_id, created_by, updated_by` | Critical | Y |
| 69 | Report | 리포트 생성 작업 | [`report_generation`](#table-report_generation) | `report_generation_id` | `project_id, requested_by, result_file_id, created_by, updated_by, document_revision_id` | High | Y |
| 70 | AI | AI 생성 작업 | [`ai_generation_job`](#table-ai_generation_job) | `ai_job_id` | `project_id, requested_by, created_by, updated_by` | High | Y |
| 71 | AI | AI 생성 결과 | [`ai_generation_result`](#table-ai_generation_result) | `ai_result_id` | `ai_job_id, selected_by, created_by, updated_by` | High | Y |
| 72 | AI | AI 생성 결과 항목 | [`ai_result_item`](#table-ai_result_item) | `ai_result_item_id` | `ai_result_id, created_by, updated_by` | High | Y |
| 73 | Regulation | 규정 근거 문서 | [`regulatory_source`](#table-regulatory_source) | `regulatory_source_id` | `file_id, verified_by, created_by, updated_by` | High | Y |
| 74 | Regulation | 규정 조항 | [`regulatory_clause`](#table-regulatory_clause) | `regulatory_clause_id` | `regulatory_source_id, created_by, updated_by` | High | Y |
| 75 | Regulation | 요구사항 규정 근거 | [`requirement_regulation`](#table-requirement_regulation) | `requirement_regulation_id` | `requirement_id, regulatory_clause_id, created_by, updated_by` | High | Y |
| 76 | Regulation | 라이브러리 규정 근거 | [`library_item_regulation`](#table-library_item_regulation) | `library_item_regulation_id` | `library_id, regulatory_clause_id, created_by, updated_by` | High | Y |

## 4. 도메인별 바로가기

- **Organization**: [`organization`](#table-organization)
- **Security**: [`app_user`](#table-app_user), [`role`](#table-role), [`user_role`](#table-user_role), [`user_group`](#table-user_group), [`user_group_member`](#table-user_group_member), [`group_role`](#table-group_role), [`access_permission_grant`](#table-access_permission_grant), [`inventory_role_grant`](#table-inventory_role_grant), [`user_signature_credential`](#table-user_signature_credential)
- **Compliance**: [`electronic_signature`](#table-electronic_signature), [`audit_trail`](#table-audit_trail)
- **File**: [`file_asset`](#table-file_asset), [`file_scan_job`](#table-file_scan_job), [`evidence_link`](#table-evidence_link), [`file_link_request`](#table-file_link_request)
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
- **Deviation**: [`deviation`](#table-deviation), [`deviation_action_round`](#table-deviation_action_round), [`deviation_completion_revision`](#table-deviation_completion_revision)
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
| 4 | 조직 유형 | `organization_type` | `varchar(50)` | N | N | - | Y | `'HEADQUARTERS'` | N | N | N | Y | HEADQUARTERS=본사, PLANT=공장, LAB=연구소, OVERSEAS=해외법인, SYSTEM=시스템 전용 조직, VENDOR=개발사 운영 조직. SYSTEM은 SYSTEM 계정이 소속되는 내부 전용 조직으로 DB당 1건이며 화면의 조직 목록·선택지에서 제외한다. 화면 표시명은 코드별 한글 라벨로 매핑한다 | `HEADQUARTERS` |
| 5 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE | `ACTIVE` |
| 6 | AI 기능 사용 여부 | `ai_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | AI 초안 생성과 감사대응 봇을 한 번에 켜고 끄는 조직 단위 스위치. FALSE이면 화면에서 두 기능의 진입점을 숨기고 서버에서 신규 AI 작업 생성과 봇 질의를 거부한다. SYSTEM 조직은 FALSE 고정 | `TRUE` |
| 7 | SSO 사용 여부 | `sso_enabled` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 조직의 로그인 방식. TRUE이면 소속 HUMAN 계정은 회사 IdP(SSO)로 로그인하고, FALSE이면 플랫폼 자체 비밀번호(LOCAL)로 로그인한다. 고객사마다 SSO 도입 여부가 다르므로 조직 단위로 지정한다. IdP 연결 정보(발급처·인증서·클라이언트 설정)는 이 표가 아닌 서버 설정에서 관리한다. SYSTEM 조직은 FALSE | `TRUE` |
| 8 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 조직 설명 | `기업 IT 및 데이터 운영 조직` |
| 9 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 10 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_organization_1` | CHECK | `status` | CHECK (status IN ('ACTIVE','INACTIVE')) |
| `ck_organization_2` | CHECK | `organization_type` | CHECK (organization_type IN ('HEADQUARTERS','PLANT','LAB','OVERSEAS','SYSTEM','VENDOR')) |
| `ck_organization_3` | CHECK | `organization_type, ai_enabled` | CHECK (organization_type <> 'SYSTEM' OR ai_enabled = FALSE) |
| `rule_organization_4` | 업무 검증 | `organization_type` | organization_type='SYSTEM'인 조직은 DB당 1건이며 account_type=SYSTEM 계정만 소속된다. 프로젝트·시스템 자산·사용자 그룹을 소유할 수 없고 화면의 조직 목록·선택지·보고서에 표시하지 않는다. |
| `rule_organization_5` | 업무 검증 | `ai_enabled` | ai_enabled 변경은 관리자 등급(SYSTEM_ADMIN 또는 해당 조직 ADMIN)만 수행하며 변경 전후 값을 Audit Trail(menu_code=ACCOUNTS)에 기록한다. |
| `rule_organization_6` | 업무 검증 | `sso_enabled` | sso_enabled 변경은 관리자 등급만 수행하며 Audit Trail(menu_code=ACCOUNTS)에 기록한다. 변경 시 소속 HUMAN 계정의 auth_provider·SSO 식별자·password_hash를 조직 단위로 일괄 전환한다(비상 접근 계정 제외). 기존 비밀번호 폐기 시점 등 세부 절차는 운영 절차로 정한다. |
| `ck_organization_7` | CHECK | `organization_type, sso_enabled` | CHECK (organization_type <> 'SYSTEM' OR sso_enabled = FALSE) |
| `rule_organization_8` | 업무 검증 | `organization_type` | organization_type='VENDOR'인 조직은 DB당 1건이며 개발사 운영 인력(permission_level=SYSTEM_ADMIN)만 소속된다. 프로젝트·시스템 자산·사용자 그룹을 소유할 수 없고 고객사 화면의 조직 목록·선택지·통계에서 제외한다. sso_enabled는 개발사 IdP 사용 여부에 따라 지정한다. |

#### 업무 규칙

사용자·그룹·시스템의 소속 조직을 관리한다.

ai_enabled는 조직 단위 AI 사용 정책이다. TRUE이면 AI 초안 생성(ai_generation_job)과 감사대응 봇을 모두 사용할 수 있고, FALSE이면 두 기능을 함께 차단한다. 기능별 개별 스위치는 두지 않는다. FALSE로 전환한 뒤에는 새 AI 작업을 생성하지 않으며, 전환 시점에 이미 PENDING/PROCESSING인 작업은 완료까지 허용한다.

SYSTEM 조직은 배치·자동 처리용 SYSTEM 계정의 소속을 나타내기 위한 내부 조직이다. 업무 데이터의 소유 주체가 아니며 고객사 화면과 통계에서 제외한다.

VENDOR 조직은 개발사 운영 인력(SYSTEM_ADMIN)이 소속되는 조직이다. 고객사 조직과 분리하여 고객사 IdP 장애와 무관하게 개발사 IdP(SSO) 또는 비상 접근(LOCAL)으로 접속한다. 업무 데이터의 소유 주체가 아니며 고객사 화면과 통계에서 제외한다.

sso_enabled는 조직의 로그인 방식을 정한다. TRUE인 조직의 사람 계정은 SSO로만 로그인하며 IdP 장애 대비 비상 접근 계정만 예외로 LOCAL을 쓴다. FALSE인 조직의 사람 계정은 모두 LOCAL로 로그인한다. 전자서명은 두 방식 모두 user_signature_credential의 서명 전용 비밀번호로 재인증한다.

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
| 11 | 사용자 ID | `user_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 계정 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 12 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | organization.organization_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 13 | 비밀번호 해시 | `password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | 플랫폼 자체 로그인(auth_provider=LOCAL) 비밀번호 검증용 해시값. auth_provider=LOCAL 계정(SSO 미사용 조직의 사용자, SSO 사용 조직의 비상 접근 계정)에서 필수이며 SSO 계정과 SYSTEM 계정은 NULL을 유지한다. 전자서명 재인증에는 사용하지 않는다(서명 비밀번호는 user_signature_credential). 응답·일반 내보내기·감사로그에 해시 원문을 포함하지 않는다. Audit=N을 유지한다. | `[REDACTED_PASSWORD_HASH]` |
| 14 | 직전 비밀번호 해시 | `previous_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | 직전 로그인 비밀번호(password_hash)의 해시값. LOCAL 계정이 비밀번호를 변경·재설정할 때 직전 비밀번호와 같은 값을 거부하는 데 사용한다(요구사항 1.3). SSO·SYSTEM 계정은 NULL. 응답·일반 내보내기·감사로그에 포함하지 않으며 Audit=N을 유지한다. | `[REDACTED_PASSWORD_HASH]` |
| 15 | 사용자 실명 | `full_name` | `varchar(100)` | N | N | - | Y | - | N | Y | Y | Y | 사용자 이름 | `홍길동` |
| 16 | 이메일 | `email` | `varchar(254)` | N | N | - | Y | - | Y | Y | Y | Y | 계정 초대·로그인에 사용하는 이메일. 앞뒤 공백 제거·소문자로 정규화한 값으로 저장한다. 화면의 계정 값이며 별도 로그인 ID를 두지 않는다. | `user@example.com` |
| 17 | 인증 방식 | `auth_provider` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 이 계정이 로그인하는 방식. SSO=회사 IdP가 본인 확인, LOCAL=플랫폼이 보관한 password_hash로 로그인. 서버가 소속 조직의 sso_enabled에 따라 지정한다: SSO 사용 조직은 SSO(예외: IdP 장애 대비 비상 접근용 SYSTEM_ADMIN 계정만 LOCAL), SSO 미사용 조직은 LOCAL. SYSTEM 계정은 로그인하지 않으므로 값과 무관 | `SSO` |
| 18 | SSO 식별자 | `sso_subject` | `varchar(255)` | N | N | - | N | - | N | Y | Y | Y | IdP가 발급한 사용자 식별자(OIDC sub / SAML NameID). 발급처(sso_issuer) 안에서만 유일하므로 (sso_issuer, sso_subject) 조합으로 계정을 매칭하며 이메일은 보조 확인에만 사용한다. auth_provider=SSO인 HUMAN 계정에서 필수, LOCAL·SYSTEM 계정은 NULL | `[REDACTED_SSO_SUBJECT]` |
| 19 | SSO 발급처 | `sso_issuer` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | sso_subject를 발급한 IdP의 식별자(OIDC issuer URL / SAML entityID). 같은 고객사 안에서도 법인·사업장별로 다른 IdP를 쓸 수 있으므로 sso_subject와 함께 저장하여 계정을 구분한다. auth_provider=SSO인 HUMAN 계정에서 필수, LOCAL·SYSTEM 계정은 NULL | `https://login.example.com/tenant-sample` |
| 20 | 계정 유형 | `account_type` | `varchar(20)` | N | N | - | Y | `'HUMAN'` | N | Y | N | Y | HUMAN=사람 사용자, SYSTEM=배치·자동 처리의 수행 주체 계정. SYSTEM은 DB당 1개이며 로그인·전자서명·결재 담당 배정에 사용하지 않고 시드·배치 처리의 created_by/updated_by/requested_by 등 FK 값으로만 사용한다 | `HUMAN` |
| 21 | 관리 권한등급 | `permission_level` | `varchar(20)` | N | N | - | Y | `'USER'` | N | N | N | Y | SYSTEM_ADMIN=개발사 관리자(전체 고객사·조직·운영 설정 관리), ADMIN=고객사 관리자(자기 조직의 계정·권한·설정 관리, 종전 GLOBAL_ADMIN), USER=일반 사용자. 종전 PROJECT_ADMIN은 화면에 해당 개념이 없어 삭제하며 프로젝트 단위 역할은 project_member로 관리한다. 작성자·검토자 등의 업무 역할(role)과 구분하며 관리자 등급은 role에 두지 않는다 | `USER` |
| 22 | 부서명 | `department_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 소속 부서명 | `정보전략팀` |
| 23 | 직급/직책 | `position_title` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 직급 정보 | `선임` |
| 24 | 계정 상태 | `status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE \| INACTIVE \| LOCKED | `ACTIVE` |
| 25 | 최종 로그인 시각 | `last_login_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 시스템 접속 타임스탬프 | `2026-08-26T16:00:00Z` |
| 26 | 로그인 연속 실패 횟수 | `failed_login_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | LOCAL 로그인 비밀번호 연속 실패 횟수. 성공 시 0으로 초기화하고 5회 연속 실패하면 status=LOCKED로 전환한다(요구사항 1.3). 시간이 지나도 자동으로 해제하지 않으며 관리자가 초기화하면 0으로 초기화한다. SSO 계정은 IdP가 실패를 관리하므로 사용하지 않는다 | `0` |
| 27 | 비밀번호 변경일 | `password_changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 로그인 비밀번호(password_hash) 마지막 변경 시각. LOCAL 계정에서만 사용하며 SSO·SYSTEM 계정은 NULL. 서명 비밀번호 변경 시각은 user_signature_credential.changed_at | `2026-08-01T09:00:00Z` |
| 28 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 29 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T16:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_app_user_1` | 업무 검증 | `email` | 이메일은 앞뒤 공백 제거·소문자로 정규화한 값으로 저장하고(ck_app_user_email) 정규화 값 기준으로 중복을 허용하지 않는다(email UQ). |
| `ck_app_user_2` | CHECK | `permission_level` | CHECK (permission_level IN ('SYSTEM_ADMIN','ADMIN','USER')) |
| `ck_app_user_3` | CHECK | `status` | CHECK (status IN ('ACTIVE','INACTIVE','LOCKED')) |
| `ck_app_user_4` | CHECK | `auth_provider` | CHECK (auth_provider IN ('SSO','LOCAL')) |
| `ck_app_user_5` | CHECK | `account_type` | CHECK (account_type IN ('HUMAN','SYSTEM')) |
| `ck_app_user_6` | CHECK | `account_type, auth_provider, sso_issuer, sso_subject, password_hash` | CHECK ((account_type='SYSTEM' AND password_hash IS NULL AND sso_issuer IS NULL AND sso_subject IS NULL) OR (account_type='HUMAN' AND auth_provider='SSO' AND sso_issuer IS NOT NULL AND sso_subject IS NOT NULL AND password_hash IS NULL) OR (account_type='HUMAN' AND auth_provider='LOCAL' AND password_hash IS NOT NULL AND sso_issuer IS NULL AND sso_subject IS NULL)) |
| `uq_app_user_7` | UNIQUE | `sso_issuer, sso_subject` | UNIQUE (sso_issuer, sso_subject) WHERE sso_subject IS NOT NULL |
| `rule_app_user_8` | 업무 검증 | `auth_provider, organization_id, permission_level` | HUMAN 계정의 auth_provider는 소속 조직의 sso_enabled와 일치해야 한다(TRUE면 SSO, FALSE면 LOCAL). 예외로 sso_enabled=TRUE 조직에서는 permission_level=SYSTEM_ADMIN인 비상 접근 계정에 한해 LOCAL을 허용한다. |
| `rule_app_user_9` | 업무 검증 | `account_type` | account_type=SYSTEM 계정은 DB당 1개이며 organization_type=SYSTEM 조직에 소속된다. permission_level=USER, status=ACTIVE를 유지하고 로그인·전자서명·결재 담당 배정·user_signature_credential 생성 대상에서 제외한다. 삭제·비활성화하지 않는다. |
| `rule_app_user_10` | 업무 검증 | `permission_level` | SYSTEM_ADMIN은 개발사 운영 인력에게만 부여하고 고객사 관리자는 ADMIN을 사용한다. SYSTEM_ADMIN 계정은 organization_type=VENDOR 조직에만 소속되며 VENDOR 조직의 HUMAN 계정은 모두 SYSTEM_ADMIN이다. 관리자 등급도 전자서명 본인 확인·담당 배정·작성/검토/승인자 분리 규칙을 우회하지 않는다. |
| `rule_app_user_11` | 업무 검증 | `auth_provider` | 비상 접근 계정(sso_enabled=TRUE 조직의 LOCAL 계정)은 전자서명과 결재 담당 배정에 사용할 수 없으며 로그인·사용 내역을 Audit Trail에 기록한다. 허용 개수와 보관·사용 절차는 운영 정책(SOP)으로 정한다(결정 대기). SSO 미사용 조직의 LOCAL 계정은 일반 사용자로서 서명·결재할 수 있다. |
| `rule_app_user_12` | 업무 검증 | `password_hash, previous_password_hash, status, failed_login_count` | LOCAL 계정은 로그인 비밀번호를 5회 연속 실패하면 status=LOCKED로 전환하고, 시간이 지나도 자동으로 해제하지 않으며 관리자 초기화로만 해제한다. 비밀번호를 변경·재설정할 때 직전 비밀번호(previous_password_hash)와 같은 값은 허용하지 않는다(요구사항 1.3). 복잡도·만료 주기는 운영 정책을 따른다(결정 대기). 연속 실패는 failed_login_count로 세며, 잠그면 ACCOUNT_LOCKED를, 비밀번호를 변경하면 PASSWORD_CHANGE를 Audit Trail에 기록한다. |
| `ck_app_user_failed_login` | CHECK | `failed_login_count` | CHECK (failed_login_count >= 0) |
| `ck_app_user_previous_password` | CHECK | `auth_provider, previous_password_hash` | CHECK (previous_password_hash IS NULL OR auth_provider = 'LOCAL') |
| `ck_app_user_email` | CHECK | `email` | CHECK (email = lower(trim(email))) |

#### 업무 규칙

복수 업무 역할은 user_role, 복수 소속 그룹은 user_group_member에서 관리한다. 화면의 활성/비활성은 ACTIVE/INACTIVE에 대응한다. 계정 잠금 상태, 비밀번호 정보 및 접속 시각을 관리한다.

로그인 방식은 소속 조직의 sso_enabled로 정한다. SSO 사용 조직의 HUMAN 계정은 auth_provider=SSO이며 IdP 인증 결과의 (발급처, 식별자)를 (sso_issuer, sso_subject)와 대조하여 계정을 매칭한다. 플랫폼은 SSO 계정의 로그인 비밀번호를 보유하지 않으며 password_hash는 NULL을 유지한다. SSO 미사용 조직의 HUMAN 계정은 auth_provider=LOCAL이며 이메일과 password_hash로 로그인한다. 화면 표시·초대 식별자는 이메일이고 SSO 매칭에서 이메일은 보조 확인에만 사용한다. 매칭되는 계정이 없거나, 비밀번호가 일치하지 않거나, 계정이 INACTIVE·LOCKED이거나 SYSTEM 계정이면 로그인을 거부하고 audit_trail에 LOGIN_FAILED로 기록한다.

SSO 사용 조직에서 auth_provider=LOCAL은 IdP 장애 등 비상 접근(break-glass)을 위한 SYSTEM_ADMIN 계정에만 허용한다. 비상 접근 계정은 password_hash로 로그인하며 전자서명·결재 담당 배정에 사용할 수 없고, 모든 로그인·사용 내역을 Audit Trail에 남긴다. 평상시에는 사용하지 않으며 존재 여부를 주기적으로 점검한다. SSO 미사용 조직의 LOCAL 계정은 비상 접근 계정이 아니며 일반 업무·서명에 사용한다.

전자서명 재인증은 로그인 방식과 무관하게 user_signature_credential의 서명 전용 비밀번호를 사용한다. 로그인 비밀번호와 서명 비밀번호를 같은 값으로 두지 않는다.

account_type=SYSTEM은 배치·자동 처리의 수행 주체를 나타내는 계정이다. DB당 1개만 두고 SYSTEM 조직에 소속시키며, 시드·배치 처리의 created_by/updated_by/requested_by 등 FK 값으로 사용한다. Audit Trail에는 actor_type=SYSTEM 또는 BATCH와 함께 이 계정의 user_id를 actor_id로 기록한다.

관리 권한등급(SYSTEM_ADMIN=개발사 관리자, ADMIN=고객사 관리자, USER=일반 사용자)과 메뉴·프로젝트별 접근권한은 별도로 관리한다. 관리자 등급은 role 테이블이 아니라 permission_level에서만 관리한다.

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
| 30 | 역할 ID | `role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 역할/권한 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 31 | 역할 코드 | `role_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | AUTHOR=작성자, REVIEWER=검토자, APPROVER=승인자, VIEWER=뷰어. 관리자 등급(SYSTEM_ADMIN/ADMIN)은 업무 역할이 아니므로 role에 두지 않고 app_user.permission_level에서 관리한다 | `AUTHOR` |
| 32 | 역할명 | `role_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 계정 화면의 업무 역할 표시명. 사용자 1명에게 여러 역할을 부여할 수 있다. | `작성자` |
| 33 | 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 역할 상세 권한 범위 설명 | `업무 항목 작성 및 상신 권한` |
| 34 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_role_1` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','VIEWER')) |

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
| 주요 참조(FK) | `user_id, role_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 35 | 매핑 ID | `user_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자-역할 매핑 고유 식별자. 활성(is_active=TRUE) 행에 (user_id, role_id) 중복을 허용하지 않는다. 역할 회수는 is_active=FALSE로 처리하며 행을 삭제하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 36 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 37 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | role.role_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 38 | 활성 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 역할 부여 활성 여부. 역할 회수 시 FALSE로 변경하며 행을 삭제하지 않는다 | `TRUE` |
| 39 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 40 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 역할을 부여한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 41 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-01T00:00:00Z` |
| 42 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 역할 부여를 최종 수정하거나 회수한 사용자 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_role_1` | UNIQUE | `user_id, role_id, is_active` | UNIQUE (user_id, role_id) WHERE is_active=TRUE |

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
| 43 | 사용자 그룹 ID | `user_group_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 44 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | 사용자 그룹이 소속된 조직 | `00000000-0000-0000-0000-000000000001` |
| 45 | 그룹 코드 | `group_code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 조직 내 사용자 그룹 식별 코드. deleted_at IS NULL이고 is_active가 TRUE인 행에 (organization_id, group_code) 중복을 허용하지 않는다. | `QA_REVIEWER_GROUP` |
| 46 | 그룹명 | `group_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 사용자에게 표시되는 그룹명 | `QA 검토자 그룹` |
| 47 | 그룹 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 그룹의 목적과 권한 범위 설명 | `Validation 문서 QA 검토 담당자 그룹` |
| 48 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 사용자 그룹 활성 여부 | `TRUE` |
| 49 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자 그룹 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 50 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 사용자 그룹을 생성한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 51 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자 그룹 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 52 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 사용자 그룹을 최종 수정한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 53 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 사용자 그룹 소프트 삭제 시각 | `2026-09-15T09:00:00Z` |

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
| 54 | 그룹 구성원 ID | `user_group_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹 구성원 매핑 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 55 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | 구성원이 소속된 사용자 그룹 | `00000000-0000-0000-0000-000000000001` |
| 56 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹에 포함되는 사용자. deleted_at IS NULL이고 member_status가 ACTIVE인 행에 (user_group_id, user_id) 중복을 허용하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 57 | 구성원 상태 | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | 그룹 구성원 상태. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 58 | 참여 시작 시각 | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 사용자가 그룹에 포함된 시각 | `2026-09-14T09:00:00Z` |
| 59 | 참여 종료 시각 | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 사용자의 그룹 참여가 종료된 시각. 현재 구성원이면 NULL | `2026-09-15T09:00:00Z` |
| 60 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 구성원 매핑 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 61 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 구성원을 등록한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 62 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 구성원 매핑 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 63 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 구성원 매핑을 최종 수정한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 64 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 그룹 구성원 매핑 소프트 삭제 시각 | `2026-09-15T09:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_group_member_1` | UNIQUE | `user_group_id, user_id, deleted_at, member_status` | UNIQUE (user_group_id, user_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |
| `ck_user_group_member_status` | CHECK | `member_status` | CHECK (member_status IN ('ACTIVE','INACTIVE','WITHDRAWN')) |

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
| 65 | 그룹 역할 ID | `group_role_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 사용자 그룹과 역할 간 권한 매핑 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 66 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | Y | - | N | Y | N | Y | 역할을 부여받는 사용자 그룹 | `00000000-0000-0000-0000-000000000001` |
| 67 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | 그룹에 부여하는 역할 | `00000000-0000-0000-0000-000000000001` |
| 68 | 권한 범위 유형 | `scope_type` | `varchar(20)` | N | N | - | Y | `'GLOBAL'` | N | Y | N | Y | 그룹 역할 적용 범위. GLOBAL, PROJECT | `PROJECT` |
| 69 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | scope_type이 PROJECT인 경우 필수. GLOBAL인 경우 NULL. 범위와 NULL 대응은 CHECK 제약으로 검증한다 | `00000000-0000-0000-0000-000000000001` |
| 70 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 그룹 역할 매핑 사용 여부. deleted_at IS NULL이고 is_active가 TRUE인 행에 범위별 중복 금지를 적용한다. GLOBAL은 (user_group_id, role_id), PROJECT는 (user_group_id, role_id, project_id)를 기준으로 한다. | `TRUE` |
| 71 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 역할 매핑 생성 시각(UTC) | `2026-09-14T09:00:00Z` |
| 72 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 역할을 부여한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 73 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 그룹 역할 매핑 최종 수정 시각(UTC) | `2026-09-14T09:00:00Z` |
| 74 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 그룹 역할 매핑을 최종 수정한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 75 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 그룹 역할 매핑 소프트 삭제 시각 | `2026-09-15T09:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_group_role_1` | CHECK | `scope_type, project_id` | CHECK ((scope_type='GLOBAL' AND project_id IS NULL) OR (scope_type='PROJECT' AND project_id IS NOT NULL)) |
| `uq_group_role_global` | UNIQUE | `user_group_id, role_id, scope_type, is_active, deleted_at` | UNIQUE (user_group_id, role_id) WHERE scope_type='GLOBAL' AND is_active=TRUE AND deleted_at IS NULL |
| `uq_group_role_project` | UNIQUE | `user_group_id, role_id, project_id, scope_type, is_active, deleted_at` | UNIQUE (user_group_id, role_id, project_id) WHERE scope_type='PROJECT' AND is_active=TRUE AND deleted_at IS NULL |

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
| 76 | 권한 ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 77 | 개인 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 78 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | user_group.user_group_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 79 | 권한 범위 | `scope_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MENU=메뉴, PROJECT=프로젝트 | `MENU` |
| 80 | 메뉴 코드 | `menu_code` | `varchar(40)` | N | N | - | N | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / AUDIT_TRAIL / ACCOUNTS. audit_trail.menu_code와 같은 코드 체계의 부분집합이며 화면 메뉴 단위 권한에만 사용한다. 규정 근거 관리는 LIBRARY 메뉴에 포함한다. MENU 범위에서 필수 | `LIBRARY` |
| 81 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | PROJECT 범위에서 필수. MENU 범위에서는 NULL | `00000000-0000-0000-0000-000000000001` |
| 82 | 조회 허용 | `can_view` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 조회 체크박스 | `FALSE` |
| 83 | 편집 허용 | `can_edit` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 편집 체크박스 | `FALSE` |
| 84 | 폐기 허용 | `can_dispose` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 프로젝트 권한 화면의 폐기 체크박스. 메뉴 범위에서는 FALSE | `FALSE` |
| 85 | 활성 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 권한 회수 시 FALSE로 변경 | `TRUE` |
| 86 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 87 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 88 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 89 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_access_permission_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_access_permission_grant_2` | CHECK | `scope_type, menu_code, project_id, can_dispose` | CHECK ((scope_type='MENU' AND menu_code IS NOT NULL AND project_id IS NULL AND can_dispose=FALSE) OR (scope_type='PROJECT' AND project_id IS NOT NULL AND menu_code IS NULL)) |
| `rule_access_permission_grant_3` | 업무 검증 | `can_edit` | can_edit 또는 can_dispose가 TRUE이면 can_view도 TRUE여야 한다. Audit Trail 메뉴(menu_code=AUDIT_TRAIL)의 can_edit는 FALSE이다. |
| `rule_access_permission_grant_4` | 업무 검증 | - | 활성 권한은 개인+메뉴, 그룹+메뉴, 개인+프로젝트, 그룹+프로젝트의 각 조합별로 중복을 허용하지 않는다. |
| `uq_access_user_id_menu` | UNIQUE | `user_id, menu_code, is_active, scope_type` | UNIQUE (user_id, menu_code) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_id_project` | UNIQUE | `user_id, project_id, is_active, scope_type` | UNIQUE (user_id, project_id) WHERE is_active=TRUE AND user_id IS NOT NULL AND scope_type='PROJECT' |
| `uq_access_user_group_id_menu` | UNIQUE | `user_group_id, menu_code, is_active, scope_type` | UNIQUE (user_group_id, menu_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='MENU' |
| `uq_access_user_group_id_project` | UNIQUE | `user_group_id, project_id, is_active, scope_type` | UNIQUE (user_group_id, project_id) WHERE is_active=TRUE AND user_group_id IS NOT NULL AND scope_type='PROJECT' |
| `ck_access_view_required` | CHECK | `can_edit, can_dispose, can_view` | CHECK ((NOT can_edit AND NOT can_dispose) OR can_view) |
| `ck_access_permission_grant_menu` | CHECK | `menu_code` | CHECK (menu_code IS NULL OR menu_code IN ('INVENTORY','PROJECT','LIBRARY','AUDIT_TRAIL','ACCOUNTS')) |

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
| 90 | 인벤토리 역할 ID | `grant_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 91 | 개인 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 92 | 사용자 그룹 ID | `user_group_id` | `uuid` | N | Y | `user_group.user_group_id` | N | - | N | Y | N | Y | user_group.user_group_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 93 | 인벤토리 역할 | `role_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | AUTHOR=작성자, REVIEWER=검토자, APPROVER=승인자, DISPOSER=폐기 권한자 | `APPROVER` |
| 94 | 활성 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 활성 여부 값 | `TRUE` |
| 95 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 96 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 97 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 98 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_inventory_role_grant_1` | CHECK | `user_id, user_group_id` | CHECK ((user_id IS NOT NULL AND user_group_id IS NULL) OR (user_id IS NULL AND user_group_id IS NOT NULL)) |
| `ck_inventory_role_grant_2` | CHECK | `role_code` | CHECK (role_code IN ('AUTHOR','REVIEWER','APPROVER','DISPOSER')) |
| `rule_inventory_role_grant_3` | 업무 검증 | - | 활성 개인+역할, 활성 그룹+역할의 각 조합별로 중복을 허용하지 않는다. |
| `uq_inventory_user_id` | UNIQUE | `user_id, role_code, is_active` | UNIQUE (user_id, role_code) WHERE is_active=TRUE AND user_id IS NOT NULL |
| `uq_inventory_user_group_id` | UNIQUE | `user_group_id, role_code, is_active` | UNIQUE (user_group_id, role_code) WHERE is_active=TRUE AND user_group_id IS NOT NULL |

#### 업무 규칙

계정의 일반 업무 역할과 별도로 화면에서 지정하는 인벤토리 권한이다. 개인 부여와 활성 그룹 상속을 합산한다.

[↑ 맨 위로](#top)

---

<a id="table-user_signature_credential"></a>
### 10. 전자서명 비밀번호 (`user_signature_credential`)

| 항목 | 정의 |
|---|---|
| 설명 | 전자서명 재인증 전용 비밀번호의 해시·상태·만료·잠금 관리. 로그인 비밀번호와 분리하여 SSO 환경에서도 서명 시 본인 확인을 수행 |
| Primary Key | `credential_id` |
| 주요 참조(FK) | `user_id, reset_requested_by, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존. 계정 비활성화 후에도 행을 삭제하지 않는다. |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 99 | 서명 자격 ID | `credential_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 100 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | Y | Y | N | Y | 서명 비밀번호의 소유 사용자. 사용자당 1행(1:1)이며 account_type=HUMAN 계정만 허용 | `00000000-0000-0000-0000-000000000001` |
| 101 | 서명 비밀번호 해시 | `signature_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | 전자서명 재인증 전용 비밀번호의 해시값. 로그인 비밀번호(app_user.password_hash)와 별도이며 같은 값을 허용하지 않는다. NOT_SET 상태에서는 NULL. 응답·내보내기·Audit JSON·signed_payload에 포함하지 않으며 Audit=N을 유지한다 | `[REDACTED_PASSWORD_HASH]` |
| 102 | 직전 서명 비밀번호 해시 | `previous_signature_password_hash` | `varchar(255)` | N | N | - | N | - | N | N | Y | N | 직전 서명 비밀번호의 해시값. 변경·재설정할 때 직전 비밀번호와 같은 값을 거부하는 데 사용한다(요구사항 1.3). 변경·재설정 이력이 없으면 NULL. 응답·내보내기·Audit JSON·signed_payload에 포함하지 않으며 Audit=N을 유지한다 | `[REDACTED_PASSWORD_HASH]` |
| 103 | 자격 상태 | `credential_status` | `varchar(20)` | N | N | - | Y | `'NOT_SET'` | N | Y | N | Y | NOT_SET=미설정(서명 불가), ACTIVE=사용 가능, LOCKED=5회 연속 실패로 잠김(관리자 초기화로만 해제), RESET_REQUIRED=관리자 재설정 요청 후 사용자 재설정 대기 | `ACTIVE` |
| 104 | 최초 설정 시각 | `set_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 사용자가 서명 비밀번호를 처음 설정한 시각 | `2026-09-01T09:00:00Z` |
| 105 | 최종 변경 시각 | `changed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 서명 비밀번호를 마지막으로 변경한 시각. 만료 계산 기준 | `2026-09-01T09:00:00Z` |
| 106 | 만료 시각 | `expires_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | changed_at에 정책 만료 주기를 더한 시각. 경과하면 ACTIVE여도 서명 불가이며 변경 후 재계산. 만료 주기 수치는 결정 대기 | `2026-12-01T09:00:00Z` |
| 107 | 연속 실패 횟수 | `failed_attempt_count` | `integer` | N | N | - | Y | `0` | N | N | N | Y | 서명 비밀번호 재인증 연속 실패 횟수. 성공 시 0으로 초기화하고 5회 연속 실패하면 LOCKED로 전환한다(요구사항 1.3). 관리자 초기화 후 새 서명 비밀번호를 설정하면 0으로 초기화한다 | `0` |
| 108 | 재설정 요청자 ID | `reset_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | RESET_REQUIRED로 전환한 관리자. 관리자는 재설정을 요청할 수만 있고 비밀번호 값을 지정할 수 없다 | `00000000-0000-0000-0000-000000000001` |
| 109 | 재설정 요청 시각 | `reset_requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | RESET_REQUIRED로 전환한 시각 | `2026-09-01T09:00:00Z` |
| 110 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 111 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 112 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 113 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_user_signature_credential_1` | UNIQUE | `user_id` | UNIQUE (user_id) |
| `ck_user_signature_credential_2` | CHECK | `credential_status` | CHECK (credential_status IN ('NOT_SET','ACTIVE','LOCKED','RESET_REQUIRED')) |
| `ck_user_signature_credential_3` | CHECK | `credential_status, signature_password_hash` | CHECK ((credential_status='NOT_SET' AND signature_password_hash IS NULL) OR (credential_status<>'NOT_SET' AND signature_password_hash IS NOT NULL)) |
| `ck_user_signature_credential_4` | CHECK | `failed_attempt_count` | CHECK (failed_attempt_count >= 0) |
| `ck_user_signature_credential_6` | CHECK | `credential_status, reset_requested_by, reset_requested_at` | CHECK (credential_status <> 'RESET_REQUIRED' OR (reset_requested_by IS NOT NULL AND reset_requested_at IS NOT NULL)) |
| `rule_user_signature_credential_7` | 업무 검증 | `user_id` | user_id는 account_type=HUMAN 계정만 허용한다. SYSTEM 계정과 비상 접근 계정(SSO 사용 조직의 LOCAL 계정)에는 행을 만들지 않는다. SSO 미사용 조직의 LOCAL 계정은 행을 만든다. |
| `rule_user_signature_credential_8` | 업무 검증 | `signature_password_hash, previous_signature_password_hash` | 설정·변경은 본인만 수행하며 현재 서명 비밀번호로 본인을 확인한다. 최초 설정(NOT_SET)과 재설정(RESET_REQUIRED)은 현재 서명 비밀번호를 쓸 수 없으므로 로그인 재인증(SSO 계정은 SSO, LOCAL 계정은 로그인 비밀번호)으로 본인을 확인한다. 관리자는 RESET_REQUIRED 전환만 가능하며 NOT_SET 상태는 전환하지 않고 유지한다. 관리자는 값을 지정하거나 조회할 수 없다. 새 비밀번호는 직전 비밀번호(previous_signature_password_hash)와 같을 수 없다(요구사항 1.3). 복잡도는 운영 정책(결정 대기)을 따른다. |
| `rule_user_signature_credential_9` | 업무 검증 | `credential_status, failed_attempt_count, expires_at` | 재인증 실패 시 failed_attempt_count를 증가시키고 5회 연속 실패하면 LOCKED로 전환한다(요구사항 1.3). 시간이 지나도 자동으로 해제하지 않으며, 관리자가 초기화(RESET_REQUIRED 전환)하면 사용자가 로그인 재인증 후 새 서명 비밀번호를 설정해 ACTIVE로 돌아가고 failed_attempt_count를 0으로 초기화한다. 성공 시 0으로 초기화한다. expires_at 경과 시 서명을 거부하고 변경을 요구한다. 만료 주기는 결정 대기. |
| `rule_user_signature_credential_10` | 업무 검증 | - | 설정·변경은 SIGNATURE_PASSWORD_SET, 관리자 재설정 요청은 SIGNATURE_PASSWORD_RESET, 재인증 실패는 SIGNATURE_FAILED, 잠금·해제는 credential_status 변경 UPDATE로 Audit Trail(menu_code=AUTH)에 기록한다. 해시·원문·입력값은 기록하지 않는다. |

#### 업무 규칙

전자서명 재인증은 이 테이블의 서명 전용 비밀번호로만 수행한다. SSO 사용 조직에서는 플랫폼이 로그인 비밀번호를 보유하지 않으므로, 전자서명의 본인 확인(21 CFR Part 11 §11.200(a)의 식별 코드+비밀번호 두 요소)을 로그인과 분리된 서명 비밀번호로 충족한다. SSO 미사용 조직도 같은 절차를 적용하여 로그인 비밀번호와 서명 비밀번호를 분리한다. 로그인 방식(SSO/LOCAL)과 무관하게 동일한 절차를 적용한다.

credential_status가 NOT_SET·LOCKED·RESET_REQUIRED이거나 expires_at이 경과한 사용자는 전자서명이 필요한 상신·검토·승인·폐기·시험 수행 확인을 수행할 수 없다. 화면은 서명 전에 설정·변경을 먼저 요구한다.

재인증 처리 순서: 서명 요청 시 서명자 user_id의 행을 잠그고 ACTIVE·미만료·미잠금을 확인한 뒤 입력값을 해시와 대조한다. 성공하면 failed_attempt_count를 0으로 초기화하고 같은 트랜잭션에서 electronic_signature 행(authentication_method=SIGNATURE_PASSWORD, authentication_result=SUCCESS)과 대상 상태 전환·감사기록을 확정한다. 실패하면 서명 행을 만들지 않고 실패 횟수 증가와 SIGNATURE_FAILED 감사기록만 저장한다.

비밀번호 원문·해시·입력값은 electronic_signature.signed_payload, audit_trail의 old_values/new_values, API 응답 본문에 포함하지 않는다. 계정이 INACTIVE·LOCKED가 되면 이 행은 유지하되 서명을 거부하고, 계정 재활성화 시 credential_status가 NOT_SET이면 NOT_SET을 유지하고(최초 설정 필요), 그 외 상태는 RESET_REQUIRED로 전환하여 재설정을 요구한다. 이때 reset_requested_by·reset_requested_at에는 재활성화를 수행한 관리자와 시각을 기록한다.

[↑ 맨 위로](#top)

---

## Compliance

<a id="table-electronic_signature"></a>
### 11. 전자서명 (`electronic_signature`)

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
| 114 | 전자서명 ID | `signature_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 전자서명 기록 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 115 | 서명자 ID | `signer_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 116 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 실제 대상 테이블명. 업무 항목 개정, 산출물 개정, 종료 요청을 구분. 인벤토리는 system_asset PK와 revision_number로 식별 | `requirement` |
| 117 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 서명이 적용된 레코드 PK값 | `00000000-0000-0000-0000-000000000001` |
| 118 | 서명 단계 | `signature_action` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | SUBMIT / REVIEW / APPROVE / REJECT / DISPOSE / CONFIG_APPLY / EXECUTE / ABORT. 서명 의미와 대상 업무를 함께 기록. ABORT는 IQ/OQ/PQ 시험 수행 중단 또는 작성 중 결과 정정 취소 | `APPROVE` |
| 119 | 서명 목적 | `signature_meaning` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 상신/검토/승인/폐기/시험 수행 확인/시험 중단·정정 취소의 업무 의미. 종료 요청은 PROJECT_CLOSE_NORMAL 또는 PROJECT_CLOSE_FORCED | `문서 최종 승인` |
| 120 | 서명 타임스탬프 | `signed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 전자서명 수행 타임스탬프 | `2026-08-26T16:30:00Z` |
| 121 | 대상 버전 | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 개정 표시 버전 또는 개정번호. 종료 요청 CLOSE-{request_version}, 일탈 조치 ACTION-{round_number}-{revision_number}, 완료보고 COMPLETE-{조치 회차}-{revision_number}, 미수행 종료 CLOSE-{조치 회차}-{revision_number}(deviation_completion_revision). 시험 수행은 회차·결과 개정을 함께 식별한다. | `v1.0` |
| 122 | 서명 대상 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | payload_schema_version을 포함한 signed_payload의 정규화 UTF-8 JSON 전체에 대한 SHA-256 소문자 64자리. 저장한 원문에서 동일하게 재계산 가능해야 한다. | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| 123 | 재인증 방식 | `authentication_method` | `varchar(50)` | N | N | - | Y | `'SIGNATURE_PASSWORD'` | N | N | N | Y | 전자서명 수행 시 서명자 본인 확인에 사용한 재인증 방식. SIGNATURE_PASSWORD(user_signature_credential의 서명 전용 비밀번호)로 고정하며 로그인 방식(SSO/LOCAL)과 무관하다 | `SIGNATURE_PASSWORD` |
| 124 | 재인증 결과 | `authentication_result` | `varchar(20)` | N | N | - | Y | `'SUCCESS'` | N | N | N | Y | 전자서명 수행 시 재인증 처리 결과. 성공한 서명만 이 테이블에 저장하므로 SUCCESS로 고정하며, 실패는 audit_trail(SIGNATURE_FAILED)과 user_signature_credential.failed_attempt_count에만 기록한다 | `SUCCESS` |
| 125 | 서명자 표시명 | `signer_name_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 서명 당시 화면에 표시한 사용자명 | `홍길동` |
| 126 | 서명자 역할 | `signer_role_snapshot` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 서명 당시 작성자/검토자/승인자 등 역할 | `승인자` |
| 127 | 서명 원문 형식 버전 | `payload_schema_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 대상 유형·서명 목적별 포함 필드와 정규화 규칙을 식별하는 버전 | `SIGN-1` |
| 128 | 서명 원문 | `signed_payload` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | 서버가 확정한 {schema_version,target,baseline_id,body,children,source_refs,files,context}. 각 대상에 없는 영역은 빈 배열 또는 NULL로 포함한다. 비밀번호·인증 비밀은 제외한다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_electronic_signature_1` | 업무 검증 | - | target_table_name과 target_record_id는 해당 대상의 실제 PK·개정을 함께 가리켜야 한다. 여러 테이블에 대한 다형 참조이며 물리 FK가 아니다. |
| `rule_electronic_signature_2` | 업무 검증 | - | 서명자와 처리자가 일치해야 하며, 서명자의 user_signature_credential이 ACTIVE·미만료·미잠금 상태에서 서명 비밀번호 재인증에 성공한 경우에만 서명 행을 생성한다. 실패한 재인증은 이 테이블에 저장하지 않고 audit_trail에 SIGNATURE_FAILED로 기록한다. |
| `rule_electronic_signature_3` | 업무 검증 | - | 서명 후 내용은 수정하지 않는다. 대상 수정은 새 개정과 새 서명으로 처리한다. |
| `ck_signature_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_signature_auth_method` | CHECK | `authentication_method` | CHECK (authentication_method = 'SIGNATURE_PASSWORD') |
| `ck_signature_auth_result` | CHECK | `authentication_result` | CHECK (authentication_result = 'SUCCESS') |
| `rule_electronic_signature_4` | 업무 검증 | `signer_id` | signer_id는 account_type=HUMAN인 활성 계정이어야 하며 비상 접근 계정(SSO 사용 조직의 LOCAL 계정)이 아니어야 한다. SYSTEM 계정과 비상 접근 계정은 서명할 수 없고, SSO 미사용 조직의 LOCAL 계정은 서명할 수 있다. |
| `ck_signature_action` | CHECK | `signature_action` | CHECK (signature_action IN ('SUBMIT','REVIEW','APPROVE','REJECT','DISPOSE','CONFIG_APPLY','EXECUTE','ABORT')) |

#### 업무 규칙

폐기·시험 수행 확인·결재선 적용은 해당 업무 기록의 서명 FK로 연결한다. 일반 결재 처리는 workflow와 approval_action을 사용한다.

서명 시 본인 확인은 로그인 세션과 별개로 매번 수행하며, SSO 로그인 여부와 무관하게 user_signature_credential의 서명 전용 비밀번호를 입력받아 재인증한다. 재인증과 서명 행 생성·대상 상태 전환·감사기록은 같은 트랜잭션으로 확정한다.

종료 서명은 project_closure_request PK와 CLOSE-{request_version}으로 식별한다. 과거 서명 원문과 해시는 보존한다.

signed_payload는 서명자가 확인한 본문, 순서가 있는 하위 절차, 근거 대상의 테이블·PK·버전·내용 해시, 서명 시 이미 존재하는 원본 첨부파일의 file_id·content_hash·크기, 프로젝트 검증 대상 baseline_id를 포함한다. 승인 원문에서 나중에 생성한 PDF 출력 파일은 이 원본 첨부 목록에 소급 추가하지 않는다. context에는 서명 목적·처리 사유를 포함하며 완료보고는 시정·예방 조치와 완료보고 원문, 미수행 종료는 종료 사유, 폐기는 폐기 사유와 원래 승인 대상 식별자, 시험 중단·정정 취소(ABORT)는 중단 사유와 대상 수행의 attempt_no·record_revision 및 중단 시점까지 입력된 절차 완료·결과 값을 포함한다. source_refs는 정확한 개정 원문을 보존한 대상만 참조한다. 반려 후 재상신으로 변경된 원문도 각 서명의 signed_payload에 별도로 남긴다.

정규화는 객체 키를 문자 코드 순으로 정렬하고 공백 없는 JSON을 UTF-8로 직렬화한다. 업무 순서가 있는 배열은 순서를 유지하고 집합형 참조·파일 목록은 식별자 순으로 정렬한다. 날짜는 UTC 표준 문자열, UUID는 소문자, NULL은 명시적으로 유지한다. 수치와 문자열 변환 규칙은 payload_schema_version별 정의를 보존한다. 서버가 대상과 하위 데이터·파일 해시를 다시 읽고 검증한 후 서명 원문 저장, 상태 전환, 감사기록을 같은 트랜잭션으로 확정한다.

승인 상태·현재 개정 표시·updated_at·결재 진행 요약·이후의 폐기 여부·invalidated_at 등 관리 필드는 원래 업무 승인 body에서 제외한다. 관리 필드 변경도 감사기록에 남기며 원래 signed_payload와 content_hash는 갱신하지 않는다. 폐기·종료 등의 별도 서명에서는 해당 처리 사유를 context에 포함한다. 동일 대상·목적·버전의 최종 승인 원문은 다시 덮어쓰거나 재승인하지 않으며 업무 원문 변경 시 새 버전 또는 새 조치 개정을 생성한다.

[↑ 맨 위로](#top)

---

<a id="table-audit_trail"></a>
### 12. Audit Trail (`audit_trail`)

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
| 129 | 감사추적 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 감사추적 레코드 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 130 | 수행자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | USER 행위자는 필수. SYSTEM/BATCH 행위자는 SYSTEM 계정(account_type=SYSTEM)의 user_id를 저장하며 SYSTEM 계정 생성 전 초기 시드에 한해 NULL 허용. ANONYMOUS는 NULL | `00000000-0000-0000-0000-000000000001` |
| 131 | 작업 유형 | `action_type` | `varchar(40)` | N | N | - | Y | - | N | Y | N | Y | CREATE, UPDATE, DELETE, EXPORT, LOGIN, LOGOUT, LOGIN_FAILED, ACCOUNT_LOCKED, PASSWORD_CHANGE, SIGNATURE_PASSWORD_SET, SIGNATURE_PASSWORD_RESET, SIGNATURE_FAILED, REPORT_GENERATE, REPORT_DOWNLOAD, REPORT_CANCEL, PROJECT_CLOSE, PROJECT_FORCE_CLOSE, SUBMIT, REVIEW, APPROVE, REJECT, CANCEL, DISPOSE, PERMISSION_CHANGE, FILE_DOWNLOAD. 결재·폐기·권한 변경의 기록 기준은 rule_audit_trail_11, 파일 다운로드의 기록 기준은 rule_audit_trail_12 | `REPORT_GENERATE` |
| 132 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 대상 실제 테이블명. 대상 행이 없는 이벤트는 NULL | `requirement` |
| 133 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 대상 PK. 로그인·로그아웃 등 대상 행이 없는 이벤트는 NULL | `00000000-0000-0000-0000-000000000001` |
| 134 | 변경 전 데이터 | `old_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 변경 전후 업무 값. 비밀번호·토큰·password_hash 제외 | `{"status": "작성중"}` |
| 135 | 변경 후 데이터 | `new_values` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 변경 전후 업무 값. 비밀번호·토큰·password_hash 제외 | `{"status": "승인완료"}` |
| 136 | 변경 사유 | `reason_for_change` | `text` | N | N | - | N | - | N | N | N | Y | 21 CFR Part 11 데이터 변경 사유 | `요구사항 오탈자 수정 및 규정 항목 보완` |
| 137 | 접속 IP 주소 | `client_ip` | `varchar(45)` | N | N | - | N | - | N | N | Y | Y | 사용자 클라이언트 IP | `192.0.2.10` |
| 138 | 발생 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 감사추적 로그 생성 시각 (UTC) | `2026-08-26T16:30:00Z` |
| 139 | 수행 주체 유형 | `actor_type` | `varchar(20)` | N | N | - | Y | `'USER'` | N | Y | N | Y | 변경 수행 주체 유형. USER, SYSTEM, BATCH, ANONYMOUS로 구분하며 USER인 경우 actor_id를 필수로 저장. ANONYMOUS는 IdP 식별자 또는 입력한 이메일이 어떤 계정에도 매칭되지 않는 로그인 실패 등 행위자를 특정할 수 없는 이벤트 | `USER` |
| 140 | 요청 ID | `request_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 하나의 화면·API 요청에서 발생한 여러 감사추적 기록을 동일 요청으로 묶기 위한 식별자 | `00000000-0000-0000-0000-000000000001` |
| 141 | 세션 ID | `session_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 변경 작업이 발생한 사용자 로그인 세션 식별자. 시스템·배치 처리 또는 세션이 없는 요청은 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 142 | 대상 문서 버전 | `target_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 표시 버전을 저장하며 일반 테이블은 NULL 허용. 프로젝트 종료 요청·종료 기록에는 전자서명과 동일한 CLOSE-n 버전을 저장한다 | `v1.0` |
| 143 | 대상 개정 순번 | `target_revision_number` | `integer` | N | N | - | N | - | N | N | N | Y | Revision 관리 문서인 경우 변경 발생 당시의 숫자형 개정 순번을 저장하며 일반 테이블은 NULL 허용 | `1` |
| 144 | 요청 경로 | `request_uri` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 변경을 발생시킨 화면 또는 API 요청 경로. 시스템·배치 처리 등 경로가 없는 경우 NULL 허용 | `/api/fds/approve` |
| 145 | 접속 클라이언트 정보 | `user_agent` | `text` | N | N | - | N | - | N | N | Y | Y | 변경 요청에 사용된 브라우저, 운영체제 또는 클라이언트 애플리케이션 정보 | `Sample-Client/1.0` |
| 146 | 메뉴 구분 | `menu_code` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | INVENTORY / PROJECT / LIBRARY / ACCOUNTS / AUDIT_TRAIL / VP / VA / QIA / URS / FDS_GROUP / FRA / DQ / IQ / OQ / PQ / VSR / DASHBOARD / AUTH / SYSTEM_PROCESS. 메뉴 권한(access_permission_grant.menu_code)과 같은 코드 체계이며 테이블별 기록 코드는 업무 규칙의 영역 매핑을 따른다. AUDIT_TRAIL은 감사기록 조회·내보내기 화면. SYSTEM_PROCESS는 사람이 아닌 행위자(actor_type=SYSTEM·BATCH)의 기록 전용이며 초기 시드·스케줄 배치·파일 검사 Worker·파일 연결 예약 처리·시스템 유지보수를 모두 포함한다. 대상 업무 영역은 target_table_name으로 식별한다 | `URS` |
| 147 | 행위자 역할 | `actor_role_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | 감사 화면의 역할 필터 및 당시 역할 표시 | `작성자` |
| 148 | 행위자 표시명 | `actor_display_snapshot` | `varchar(100)` | N | N | - | N | - | N | N | Y | Y | 감사 화면에 표시하는 당시 사용자명 | `홍길동` |
| 149 | 변경 항목 | `field_path` | `text` | N | N | - | N | - | N | N | N | Y | 필드별 변경 화면에서 표시할 컬럼/항목 경로. 전체 이벤트면 NULL | `requirement_text` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_audit_trail_1` | 업무 검증 | - | 대상 테이블과 ID는 함께 존재하거나 함께 NULL이다. |
| `rule_audit_trail_2` | 업무 검증 | `actor_id, actor_type` | USER 행위자는 actor_id 필수. SYSTEM/BATCH 행위자는 SYSTEM 계정의 user_id를 저장하며 SYSTEM 계정 생성 전의 초기 시드에 한해 NULL을 허용한다. ANONYMOUS는 actor_id NULL이며 LOGIN_FAILED 등 행위자를 특정할 수 없는 이벤트에만 사용한다. |
| `rule_audit_trail_3` | 업무 검증 | - | 감사 기록은 추가 전용이며 화면에서 수정·삭제하지 않는다. |
| `ck_audit_trail_4` | CHECK | `action_type` | CHECK (action_type IN ('CREATE','UPDATE','DELETE','EXPORT','LOGIN','LOGOUT','LOGIN_FAILED','ACCOUNT_LOCKED','PASSWORD_CHANGE','SIGNATURE_PASSWORD_SET','SIGNATURE_PASSWORD_RESET','SIGNATURE_FAILED','REPORT_GENERATE','REPORT_DOWNLOAD','REPORT_CANCEL','PROJECT_CLOSE','PROJECT_FORCE_CLOSE','SUBMIT','REVIEW','APPROVE','REJECT','CANCEL','DISPOSE','PERMISSION_CHANGE','FILE_DOWNLOAD')) |
| `ck_audit_trail_5` | CHECK | `actor_type` | CHECK (actor_type IN ('USER','SYSTEM','BATCH','ANONYMOUS')) |
| `ck_audit_trail_6` | CHECK | `menu_code` | CHECK (menu_code IN ('INVENTORY','PROJECT','LIBRARY','ACCOUNTS','AUDIT_TRAIL','VP','VA','QIA','URS','FDS_GROUP','FRA','DQ','IQ','OQ','PQ','VSR','DASHBOARD','AUTH','SYSTEM_PROCESS')) |
| `ck_audit_trail_7` | CHECK | `actor_type, actor_id` | CHECK ((actor_type='USER' AND actor_id IS NOT NULL) OR (actor_type='ANONYMOUS' AND actor_id IS NULL) OR actor_type IN ('SYSTEM','BATCH')) |
| `rule_audit_trail_8` | 업무 검증 | `action_type, menu_code` | LOGIN/LOGOUT/LOGIN_FAILED/ACCOUNT_LOCKED/PASSWORD_CHANGE/SIGNATURE_PASSWORD_SET/SIGNATURE_PASSWORD_RESET/SIGNATURE_FAILED는 menu_code=AUTH로 기록한다. 단 SYSTEM·BATCH 행위자가 수행한 자동 처리는 SYSTEM_PROCESS로 기록한다(ck_audit_trail_10). LOGIN_FAILED는 시도한 식별자(sso_issuer·sso_subject 또는 이메일)와 거부 사유 코드를 new_values에 저장하고 비밀번호 원문·해시는 저장하지 않는다. |
| `rule_audit_trail_9` | 업무 검증 | `menu_code` | USER·ANONYMOUS 행위의 menu_code는 업무 규칙의 영역 매핑에 따라, SYSTEM·BATCH 행위는 SYSTEM_PROCESS로 서버가 지정하며 요청 값으로 정하지 않는다. 매핑에 없는 테이블을 추가할 때는 매핑을 함께 정의한다. |
| `ck_audit_trail_10` | CHECK | `actor_type, menu_code` | CHECK ((actor_type IN ('SYSTEM','BATCH')) = (menu_code = 'SYSTEM_PROCESS')) |
| `rule_audit_trail_11` | 업무 검증 | `action_type, target_table_name, target_record_id` | 결재 처리(approval_action)가 생기면 결재 대상 업무 행을 target으로 같은 이름의 action_type(SUBMIT·REVIEW·APPROVE·REJECT·CANCEL, 회수는 CANCEL)을 기록한다. 폐기가 완료되면(lifecycle_status=DISPOSED 또는 is_disposed=TRUE 전환) 대상 행에 DISPOSE를 기록한다. 역할·권한의 부여·변경·회수(user_role, group_role, user_group_member, access_permission_grant, inventory_role_grant)는 CREATE·UPDATE 대신 PERMISSION_CHANGE로 기록한다. |
| `rule_audit_trail_12` | 업무 검증 | `action_type, target_table_name, target_record_id, menu_code` | 사용자가 저장된 파일(첨부·증적·FDS/DDS 파일·VA 첨부·규정 원문 등 file_asset)을 내려받으면 FILE_DOWNLOAD를 target_table_name=file_asset, target_record_id=file_id로 기록한다. menu_code는 영역 매핑(파일은 연결 대상 업무의 코드)을 따른다. 리포트 생성 결과(report_generation) 다운로드는 REPORT_DOWNLOAD, 데이터 내보내기는 EXPORT로 기록하며 미리보기는 기록하지 않는다. |

#### 업무 규칙

LOGIN_FAILED는 플랫폼이 로그인을 거부한 이벤트다. SSO는 IdP 인증은 통과했으나 플랫폼이 거부한 경우, LOCAL은 비밀번호 불일치 또는 플랫폼이 거부한 경우다. 매칭되는 계정이 없으면 actor_type=ANONYMOUS·actor_id NULL로, 계정은 있으나 비밀번호 불일치·INACTIVE·LOCKED·SYSTEM 계정이면 actor_type=USER·해당 actor_id로 기록한다. ACCOUNT_LOCKED는 app_user.status가 LOCKED로 바뀔 때, PASSWORD_CHANGE는 LOCAL 계정의 로그인 비밀번호(password_hash)를 변경할 때 기록한다.

menu_code=SYSTEM_PROCESS는 사람이 아닌 행위자(actor_type=SYSTEM·BATCH)가 남기는 모든 기록에 사용한다. 초기 데이터 시드, 스케줄 배치, 파일 검사 Worker의 검사 상태 전환, 파일 연결 예약(file_link_request)에 따른 자동 연결, 자동 재시도가 모두 해당한다. 이때 업무 영역은 target_table_name·target_record_id로 식별하고, 사용자가 시작한 원래 화면은 같은 대상의 사용자 감사 기록(예: 업로드 완료 기록)으로 이어서 찾는다. 사용자가 화면에서 수행한 변경(USER)과 ANONYMOUS 이벤트에는 SYSTEM_PROCESS를 사용하지 않는다(ck_audit_trail_10).

영역 매핑(USER·ANONYMOUS 행위 기준. SYSTEM·BATCH 행위는 SYSTEM_PROCESS): 계정·그룹·권한(app_user, role, user_role, user_group, user_group_member, group_role, access_permission_grant, inventory_role_grant)과 organization은 ACCOUNTS, user_signature_credential과 로그인·서명 인증 이벤트는 AUTH, system_asset·system_asset_revision은 INVENTORY, library_item·regulatory_source·regulatory_clause·library_item_regulation은 LIBRARY, validation_project·project_system_baseline·project_member·project_activity·project_closure_request·project_workflow_config와 활동 마스터(validation_activity·activity_dependency)는 PROJECT로 기록한다. 업무 항목·문서는 해당 활동 코드(VP·VA·QIA·URS·FDS_GROUP·FRA·DQ·IQ·OQ·PQ·VSR)를 사용하며 requirement_regulation은 URS, fds_spec·dds_spec은 FDS_GROUP이다. 결재(workflow_instance·workflow_step·workflow_step_assignee·approval_action)는 결재 대상 업무의 영역 코드(인벤토리는 INVENTORY, 프로젝트 종료 요청은 PROJECT)를 따른다. 일탈(deviation·deviation_action_round·deviation_completion_revision)은 원본 수행의 IQ·OQ·PQ, 증적(evidence_link)과 파일(file_asset·file_link_request)은 연결 대상 업무의 코드(프로젝트 외 업로드는 LIBRARY), 업로드 완료 기록(file_scan_job)은 연결 대상이 정해지기 전이므로 file_scan_job.project_id가 있으면 PROJECT, 없으면(규정 원문 등 프로젝트 외 업로드) LIBRARY, traceability_link는 연결을 저장한 화면의 코드(RTM 화면은 DASHBOARD)를 사용한다. deliverable_document·deliverable_revision·deliverable_section은 해당 활동 코드, report_generation은 DELIVERABLE이면 대상 산출물의 활동 코드·AUDIT_TRAIL이면 AUDIT_TRAIL, AI 작업·결과(ai_generation_job·ai_generation_result·ai_result_item)는 생성 대상 업무의 활동 코드를 사용한다. electronic_signature는 서명 대상(target_table_name)의 영역 코드를 따른다(예: 결재선 적용은 PROJECT, 인벤토리는 INVENTORY, 일탈 완료·종료는 원본 수행의 IQ·OQ·PQ). 감사기록 조회·내보내기는 AUDIT_TRAIL이다.

전자서명 비밀번호 관련 이벤트는 대상 테이블을 user_signature_credential로, 대상 레코드를 해당 credential_id로 기록하며 old_values/new_values에 해시·원문을 포함하지 않는다.

[↑ 맨 위로](#top)

---

## File

<a id="table-file_asset"></a>
### 13. 파일 자산 (`file_asset`)

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
| 150 | 파일 ID | `file_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 첨부파일 메타데이터 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 151 | 업로드자 ID | `uploader_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 사용자 업로드 파일은 업로드자, 서버가 생성한 파일은 생성을 요청한 사용자, 정기 배치 생성 파일은 SYSTEM 계정의 user_id를 저장한다 | `00000000-0000-0000-0000-000000000001` |
| 152 | 원본 파일명 | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 업로드 당시 파일명 | `sample_document.pdf` |
| 153 | 저장 파일경로 | `stored_file_path` | `text` | N | N | - | Y | - | N | N | N | Y | 스토리지 저장 경로/S3 Key | `documents/2026/08/sample_001.pdf` |
| 154 | 파일 구분 | `file_category` | `varchar(30)` | N | N | - | Y | `'ATTACHMENT'` | N | Y | N | Y | 파일 업무 구분. ATTACHMENT, EVIDENCE, REPORT, EXPORT | `REPORT` |
| 155 | 파일 용량 | `file_size_bytes` | `bigint` | N | N | - | Y | `0` | N | N | N | Y | 파일 크기 (Byte) | `1048576` |
| 156 | MIME 타입 | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 첨부 파일 형식. 미확인 시 NULL 허용 | `application/pdf` |
| 157 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T16:30:00Z` |
| 158 | 파일 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | 저장 완료한 파일 원본 바이트의 SHA-256 소문자 64자리. 서버가 계산·검증한다. | - |
| 159 | 저장소 객체 버전 | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | 버전형 저장소의 정확한 객체 버전 ID. 버전 기능이 없으면 NULL이며 stored_file_path의 덮어쓰기를 금지한다. | - |
| 160 | 파일 생성 출처 | `file_origin` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | UPLOAD / SYSTEM_GENERATED. 서버가 실제 생성 경로로 지정하며 사용자 업로드는 검사 통과 및 검사 작업 연결 필수. file_category와 구분한다 | `UPLOAD` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_file_hash` | CHECK | `content_hash` | CHECK (content_hash ~ '^[0-9a-f]{64}$') |
| `ck_file_size` | CHECK | `file_size_bytes` | CHECK (file_size_bytes >= 0) |
| `ck_file_origin` | CHECK | `file_origin` | CHECK (file_origin IN ('UPLOAD','SYSTEM_GENERATED')) |
| `rule_file_asset_scan` | 업무 검증 | `file_origin, file_id` | UPLOAD는 위협 미검출 완료한 file_scan_job의 result_file_id와 연결하여 등록하고 해시·크기·업로드자·파일명·파일 구분의 일치를 검증한다. |
| `rule_file_asset_origin` | 업무 검증 | `file_origin` | SYSTEM_GENERATED는 서버 내부 생성 경로가 확인된 경우에만 허용하며 사용자 입력의 출처·파일 구분으로 검사 생략을 결정하지 않는다. |
| `ck_file_category` | CHECK | `file_category` | CHECK (file_category IN ('ATTACHMENT','EVIDENCE','REPORT','EXPORT')) |

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
### 14. 파일 악성코드 검사 작업 (`file_scan_job`)

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
| 161 | 검사 작업 ID | `scan_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 검사 한 회차의 고유 식별자. 큐 전달·결과 수신의 중복 처리 방지 기준 | `00000000-0000-0000-0000-000000000001` |
| 162 | 업로드 ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 서버가 발급한 한 업로드의 논리 식별자. 같은 원본의 재시도 회차는 동일 값을 유지하며 별도 테이블 FK가 아니다 | `00000000-0000-0000-0000-000000000001` |
| 163 | 검사 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 최초 검사는 1. 재시도·재검사는 같은 upload_id의 다음 회차로 추가하며 메시지 중복 수신만으로 증가시키지 않는다 | `1` |
| 164 | 이전 검사 작업 ID | `previous_scan_job_id` | `uuid` | N | Y | `file_scan_job.scan_job_id` | N | - | Y | Y | N | Y | 재시도·재검사의 직전 회차. 최초 회차는 NULL이며 직전 작업에서 여러 후속 회차가 갈라지지 않는다 | `00000000-0000-0000-0000-000000000001` |
| 165 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 첨부이면 해당 프로젝트. 규정 원문 등 프로젝트 외 업로드는 NULL을 허용하며 별도 메뉴·자료 접근권한을 검증한다 | `00000000-0000-0000-0000-000000000001` |
| 166 | 업로드자 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 원본 업로드를 수행한 사용자. 자동 재시도에서도 최초 업로드자를 유지한다 | `00000000-0000-0000-0000-000000000001` |
| 167 | 원본 파일명 | `original_file_name` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 업로드 당시 파일명. 파일 자산 등록 시 동일 값을 사용한다 | `design_document.pdf` |
| 168 | 격리 저장소명 | `storage_bucket` | `varchar(255)` | N | N | - | Y | - | N | Y | N | Y | 검사 원본이 저장된 버킷 또는 저장소 식별자. 접속 비밀정보와 서명 URL은 저장하지 않는다 | `upload-quarantine-example` |
| 169 | 격리 파일경로 | `stored_file_path` | `text` | N | N | - | Y | - | N | Y | N | Y | 해당 업로드에 고유한 격리 객체 경로. 업무용 다운로드 경로로 노출하지 않는다 | `uploads/sample-upload/design_document.pdf` |
| 170 | 격리 객체 버전 | `storage_version_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | 검사한 정확한 객체 버전. 버전 미지원 저장소는 NULL이며 원본 경로의 덮어쓰기를 금지한다 | - |
| 171 | 원본 내용 해시 | `content_hash` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | 서버 또는 신뢰된 검사 처리에서 확인한 원본 바이트의 SHA-256 소문자 64자리. 확인 전 NULL이며 위협 미검출 완료 시 필수 | - |
| 172 | 파일 용량 | `file_size_bytes` | `bigint` | N | N | - | Y | - | N | N | N | Y | 저장 완료한 원본의 실제 바이트 크기 | `1048576` |
| 173 | MIME 타입 | `mime_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 확인한 원본 파일 형식. 사용자 입력값만으로 파일 유형을 신뢰하지 않는다 | `application/pdf` |
| 174 | 파일 구분 | `file_category` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ATTACHMENT / EVIDENCE / REPORT / EXPORT. 업무 목적 구분이며 검사 생략 근거로 사용하지 않는다 | `ATTACHMENT` |
| 175 | 작업 상태 | `job_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED. COMPLETED는 검사가 종료되었다는 뜻이며 사용 허용 여부는 scan_result로 판단한다 | `PENDING` |
| 176 | 검사 결과 | `scan_result` | `varchar(30)` | N | N | - | N | - | N | Y | N | Y | NO_THREATS_FOUND / THREATS_FOUND / UNSCANNABLE. 위협 미검출·위협 발견·검사 불가를 구분하며 완료 전 또는 기술 실패·취소이면 NULL | `NO_THREATS_FOUND` |
| 177 | 검사 서비스·엔진명 | `scanner_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 해당 회차에 지정한 검사 서비스 또는 엔진의 식별자. 특정 제품에 종속하지 않는다 | `MALWARE_SCANNER` |
| 178 | 검사 엔진 버전 | `scanner_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 실제 사용한 엔진 버전. 검사 서비스가 제공하지 않으면 NULL을 허용한다 | - |
| 179 | 탐지 정보 버전 | `signature_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 검사에 사용한 악성코드 탐지 정보의 버전. 제공되지 않으면 NULL을 허용한다 | - |
| 180 | 검사 서비스 요청 ID | `scanner_request_id` | `varchar(255)` | N | N | - | N | - | N | Y | N | Y | 검사 서비스가 발급한 실행·요청 식별자. 결과를 작업·원본 객체와 대조할 때 사용한다 | - |
| 181 | 검사 상세 결과 | `scan_details` | `jsonb` | N | N | - | N | - | N | N | Y | Y | provider_result, reason_code, threat_names, report_reference 등 확인 가능한 검사 근거. 비밀번호·토큰·서명 URL·파일 본문은 제외한다 | `{"provider_result":"NO_THREATS_FOUND","threat_names":[]}` |
| 182 | 검사 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 검사 회차를 요청한 주체. 업로드 직후 자동 등록하는 최초 회차는 업로드자(uploaded_by), 수동 재검사는 요청 사용자, 자동 재시도는 SYSTEM 계정의 user_id를 저장한다 | `00000000-0000-0000-0000-000000000001` |
| 183 | 검사 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 해당 검사 회차를 등록한 시각 | `2026-09-18T01:00:00Z` |
| 184 | 검사 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 회차의 검사를 시작한 시각. 실행 전 실패·취소이면 NULL 가능 | `2026-09-18T01:00:05Z` |
| 185 | 검사 종료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 완료·실패·취소를 확정한 시각. 파일 자산 등록 시각과 구분한다 | `2026-09-18T01:00:20Z` |
| 186 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 작업 상태·검사 결과 또는 결과 파일 연결의 최종 변경 시각 | `2026-09-18T01:00:21Z` |
| 187 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | FAILED의 기술 오류 코드. 위협 발견은 기술 오류가 아니며 scan_result로 기록한다 | `SCAN_TIMEOUT` |
| 188 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | Y | Y | 기술 실패 상세 사유. 비밀정보·서명 URL·파일 본문을 포함하지 않는다 | `검사 응답시간 초과` |
| 189 | 등록 파일 ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | Y | Y | N | Y | 위협 미검출 후 원본 동일성을 확인하여 등록한 파일 자산. 등록 전에는 NULL이며 같은 업로드에서 한 회차만 연결한다 | `00000000-0000-0000-0000-000000000001` |

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
### 15. 증적 파일 연결 (`evidence_link`)

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
| 190 | 증적 연결 ID | `evidence_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서·시험 항목과 증적 파일 간 연결 고유 식별자. deleted_at IS NULL인 행에 (project_id, file_id, target_entity_type, target_entity_id, evidence_type) 중복을 허용하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 191 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 증적 연결이 속한 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 192 | 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | Y | - | N | Y | N | Y | 연결되는 증적 파일 ID | `00000000-0000-0000-0000-000000000001` |
| 193 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FDS_SPEC / DDS_SPEC / VP_PLAN / QIA_MODULE_ITEM / VENDOR_AUDIT / DQ_ITEM / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / IQ_EXECUTION / OQ_EXECUTION / PQ_EXECUTION / IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION / VSR_ASSESSMENT / DEVIATION / DELIVERABLE_REVISION | `IQ_STEP_EXECUTION` |
| 194 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | target_entity_type에 해당하는 테이블의 PK값. 다형 참조이므로 물리 FK는 설정하지 않음 | `00000000-0000-0000-0000-000000000001` |
| 195 | 증적 유형 | `evidence_type` | `varchar(50)` | N | N | - | Y | `'TEST_RESULT'` | N | Y | N | Y | TEST_RESULT, SCREENSHOT, LOG, REPORT, APPROVAL_DOCUMENT | `TEST_RESULT` |
| 196 | 증적 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 증적 파일의 내용 및 연결 목적 | `IQ 수행 결과 화면 캡처` |
| 197 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 198 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 생성한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 199 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 증적 연결 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 200 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 증적 연결을 최종 수정한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 201 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 증적 연결 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_evidence_link_1` | UNIQUE | `project_id, file_id, target_entity_type, target_entity_id, evidence_type, deleted_at` | UNIQUE (project_id,file_id,target_entity_type,target_entity_id,evidence_type) WHERE deleted_at IS NULL |
| `rule_evidence_link_2` | 업무 검증 | - | 대상 유형별 실제 PK·동일 프로젝트·정확한 개정을 검사한다. 다형 참조를 물리 FK로 표시하지 않는다. |
| `rule_evidence_link_3` | 업무 검증 | - | 절차별 첨부는 각 IQ/OQ/PQ_STEP_EXECUTION에, 회차 전체 첨부는 각 EXECUTION에 연결한다. 승인된 대상의 첨부는 덮어쓰지 않는다. |
| `rule_evidence_link_4` | 업무 검증 | `file_id` | file_asset의 등록 조건을 충족한 파일만 연결한다. UPLOAD이면 위협 미검출 검사와 등록 파일의 원본 동일성을 확인하며 격리 경로·검사 작업 ID를 file_id 대신 사용하지 않는다. |
| `ck_evidence_link_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','VP_PLAN','QIA_MODULE_ITEM','VENDOR_AUDIT','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','IQ_EXECUTION','OQ_EXECUTION','PQ_EXECUTION','IQ_STEP_EXECUTION','OQ_STEP_EXECUTION','PQ_STEP_EXECUTION','VSR_ASSESSMENT','DEVIATION','DELIVERABLE_REVISION')) |
| `ck_evidence_link_type` | CHECK | `evidence_type` | CHECK (evidence_type IN ('TEST_RESULT','SCREENSHOT','LOG','REPORT','APPROVAL_DOCUMENT')) |

#### 업무 규칙

VA/FDS/DDS의 대표 첨부는 해당 테이블의 file_id가 원본이다. 동일 대표 첨부를 여기서 별도 원본으로 중복 편집하지 않는다.

IQ/OQ/PQ 절차 증적을 새로 업로드하면 file_link_request(EVIDENCE)로 먼저 예약하고, 파일 검사를 통과하면 그 예약에 따라 이 테이블의 행을 만든다. 개정 시 기존 증적을 새 개정에 이어 붙이거나 이미 등록된 파일을 다른 대상에 붙이는 경우는 기존 file_id로 직접 연결한다.

[↑ 맨 위로](#top)

---

<a id="table-file_link_request"></a>
### 16. 파일 연결 예약 (`file_link_request`)

| 항목 | 정의 |
|---|---|
| 설명 | 업로드한 파일을 검사 통과 후 어느 업무 대상에 연결할지 먼저 기록하는 예약. 파일 검사 Worker가 통과 시 자동으로 연결 |
| Primary Key | `link_request_id` |
| 주요 참조(FK) | `project_id, file_id, evidence_link_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 202 | 예약 ID | `link_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 파일 연결 예약 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 203 | 업로드 ID | `upload_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | file_scan_job.upload_id. 재검사하면 scan_job_id가 바뀌므로 검사 회차가 아닌 업로드 기준으로 예약한다. 테이블 키가 아니므로 물리 FK를 두지 않는다 | `00000000-0000-0000-0000-000000000001` |
| 204 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연결 대상이 속한 프로젝트 | `00000000-0000-0000-0000-000000000001` |
| 205 | 연결 방식 | `link_kind` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | EVIDENCE=evidence_link 행 생성, PRIMARY_FILE=대상 행의 file_id 채움 | `EVIDENCE` |
| 206 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | EVIDENCE: IQ_STEP_EXECUTION / OQ_STEP_EXECUTION / PQ_STEP_EXECUTION. PRIMARY_FILE: FDS_SPEC / DDS_SPEC / VENDOR_AUDIT | `IQ_STEP_EXECUTION` |
| 207 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 행 PK. 다형 참조이므로 물리 FK를 두지 않는다(evidence_link와 같은 방식) | `00000000-0000-0000-0000-000000000001` |
| 208 | 증적 유형 | `evidence_type` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | EVIDENCE일 때 필수이며 evidence_link.evidence_type과 같은 값. PRIMARY_FILE은 NULL | `TEST_RESULT` |
| 209 | 증적 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | EVIDENCE일 때만 사용하며 연결 시 evidence_link.description으로 옮긴다. PRIMARY_FILE은 NULL | `IQ 절차 3 결과 화면 캡처` |
| 210 | 예약 상태 | `request_status` | `varchar(20)` | N | N | - | Y | `'WAITING'` | N | Y | N | Y | WAITING=검사 대기, LINKED=연결 완료, BLOCKED=검사에서 차단(file_asset 없음), FAILED=파일은 통과했으나 연결 실패(대상 잠김 등), CANCELLED=사용자 취소 또는 대상 초안 삭제 | `WAITING` |
| 211 | 연결 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 검사를 통과해 연결한 파일. LINKED에서 필수 | `00000000-0000-0000-0000-000000000001` |
| 212 | 생성 증적 연결 ID | `evidence_link_id` | `uuid` | N | Y | `evidence_link.evidence_link_id` | N | - | N | Y | N | Y | EVIDENCE로 연결해 만든 evidence_link 행. EVIDENCE이고 LINKED이면 필수 | `00000000-0000-0000-0000-000000000001` |
| 213 | 연결 완료 시각 | `linked_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 업무 대상에 연결을 완료한 시각. LINKED에서 필수 | `2026-09-01T10:05:00Z` |
| 214 | 실패 사유 코드 | `failure_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 차단·실패 사유 코드. 예: THREATS_FOUND, ENCRYPTED, FORMAT_MISMATCH, TARGET_LOCKED. BLOCKED·FAILED에서 필수 | - |
| 215 | 실패 사유 | `failure_message` | `text` | N | N | - | N | - | N | N | N | Y | 차단·실패 사유 설명 | - |
| 216 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 예약 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 217 | 예약자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 예약한 사용자. 해당 파일을 업로드한 사용자와 같아야 한다 | `00000000-0000-0000-0000-000000000001` |
| 218 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 예약 최종 수정 시각(UTC) | `2026-09-01T10:05:00Z` |
| 219 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 예약 상태를 마지막으로 바꾼 주체. Worker가 바꾸면 SYSTEM 계정의 user_id | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_file_link_request_upload` | UNIQUE | `upload_id, request_status` | UNIQUE (upload_id) WHERE request_status <> 'CANCELLED' |
| `ck_file_link_kind` | CHECK | `link_kind` | CHECK (link_kind IN ('EVIDENCE','PRIMARY_FILE')) |
| `ck_file_link_target` | CHECK | `link_kind, target_entity_type` | CHECK ((link_kind='EVIDENCE' AND target_entity_type IN ('IQ_STEP_EXECUTION','OQ_STEP_EXECUTION','PQ_STEP_EXECUTION')) OR (link_kind='PRIMARY_FILE' AND target_entity_type IN ('FDS_SPEC','DDS_SPEC','VENDOR_AUDIT'))) |
| `ck_file_link_evidence_type` | CHECK | `link_kind, evidence_type, description` | CHECK ((link_kind='EVIDENCE' AND evidence_type IN ('TEST_RESULT','SCREENSHOT','LOG','REPORT','APPROVAL_DOCUMENT')) OR (link_kind='PRIMARY_FILE' AND evidence_type IS NULL AND description IS NULL)) |
| `ck_file_link_status` | CHECK | `request_status` | CHECK (request_status IN ('WAITING','LINKED','BLOCKED','FAILED','CANCELLED')) |
| `ck_file_link_status_state` | CHECK | `request_status, link_kind, file_id, evidence_link_id, linked_at, failure_code` | CHECK ((request_status='WAITING' AND file_id IS NULL AND linked_at IS NULL AND failure_code IS NULL) OR (request_status='LINKED' AND file_id IS NOT NULL AND linked_at IS NOT NULL AND (link_kind<>'EVIDENCE' OR evidence_link_id IS NOT NULL)) OR (request_status='BLOCKED' AND file_id IS NULL AND failure_code IS NOT NULL) OR (request_status='FAILED' AND failure_code IS NOT NULL) OR (request_status='CANCELLED' AND linked_at IS NULL)) |
| `ck_file_link_evidence_link` | CHECK | `link_kind, evidence_link_id` | CHECK (evidence_link_id IS NULL OR link_kind='EVIDENCE') |
| `rule_file_link_request_1` | 업무 검증 | `created_by, upload_id, target_entity_type, target_entity_id` | 예약자는 해당 업로드를 올린 사용자(file_scan_job.uploaded_by)와 같아야 한다. 연결 대상은 존재하고 같은 프로젝트에 속하며 수정 가능한 상태여야 하고 예약자에게 대상 편집 권한이 있어야 한다. 수정 가능한 상태는 FDS·DDS·VA는 상신(REVIEW) 전, IQ/OQ/PQ 절차 수행은 상위 수행(*_execution)의 execution_status가 IN_PROGRESS일 때(결과 등록 전)이다. |
| `rule_file_link_request_2` | 업무 검증 | `request_status` | 연결 처리는 예약 행을 잠근 뒤 상태를 바꾼다. 예약 시점에 이미 검사를 통과했으면 즉시 연결하고, 아니면 Worker가 검사 통과 후 연결한다. 파일 등록(file_asset)과 업무 연결은 트랜잭션을 나눠, 연결이 실패해도 검사를 통과한 파일은 유지하고 FAILED와 사유를 남긴다. |
| `rule_file_link_request_3` | 업무 검증 | `request_status` | 사용자가 취소하면 WAITING·BLOCKED·FAILED 예약을 CANCELLED로 바꾼다. FDS·DDS 초안을 삭제(deleted_at)하거나 IQ/OQ/PQ 절차 수행의 상위 수행이 중단(ABORTED)되면 그 대상의 미연결(WAITING·BLOCKED·FAILED) 예약을 같은 트랜잭션에서 CANCELLED로 바꾼다. LINKED 예약은 취소하지 않는다. |
| `rule_file_link_request_4` | 업무 검증 | `target_entity_type, target_entity_id, request_status` | 대상의 상신·결과 등록 전에 WAITING·BLOCKED·FAILED 예약이 있는지 확인하고, 있으면 진행을 허용하지 않는다. BLOCKED·FAILED는 사용자가 취소하거나 다시 업로드해 해소한다. |

#### 업무 규칙

사용자는 파일 업로드 완료까지만 기다리고 악성코드·형식 검사는 Worker가 별도로 처리한다. 검사를 통과한 파일만 file_asset으로 등록한다는 원칙(rule_file_asset_scan)을 유지하기 위해 업로드와 연결 대상을 이 테이블에 먼저 기록하고, 검사를 통과하면 Worker가 EVIDENCE는 evidence_link 행을 만들고(created_by는 예약자) PRIMARY_FILE은 fds_spec·dds_spec·vendor_audit의 file_id를 채운다.

예약 등록·취소의 감사 기록은 화면의 menu_code(USER)로, Worker의 상태 전환과 연결은 SYSTEM_PROCESS(SYSTEM·BATCH)로 남긴다. vendor_audit.file_id가 비어 있을 때 '첨부 없음'과 '검사 중'은 이 테이블의 예약 상태로 구분한다.

개정 시 기존 증적을 새 개정에 이어 붙이거나 이미 등록된 파일을 다른 대상(일탈 등)에 붙이는 경우는 업로드가 없으므로 이 테이블을 쓰지 않고 기존 file_id로 직접 연결한다. 업로드만 하고 예약하지 않은 파일의 정리 방식은 별도로 정한다.

[↑ 맨 위로](#top)

---

## System

<a id="table-system_asset"></a>
### 17. 시스템/장비 식별 정보 (`system_asset`)

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
| 220 | 시스템 ID | `system_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 시스템 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 221 | 조직 ID | `organization_id` | `uuid` | N | Y | `organization.organization_id` | Y | - | N | Y | N | Y | 시스템/장비가 소속된 고객사 또는 운영 조직 | `00000000-0000-0000-0000-000000000001` |
| 222 | 관리번호 | `management_number` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 자산/설비 관리번호 | `EQ-MES-2024-001` |
| 223 | 시스템명 | `system_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 시스템/장비명 | `Sample System` |
| 224 | 대분류 | `major_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 컴퓨터화 시스템/장비/시설·유틸리티 | `컴퓨터화 시스템` |
| 225 | 중분류 | `middle_category` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 선택 대분류에 속하는 중분류 | `품질 시스템` |
| 226 | 대상 | `target_type` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 선택 대분류·중분류에 속하는 대상 | `LIMS` |
| 227 | 담당부서 | `department_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 관리/운용 담당부서 | `생산기술팀` |
| 228 | 설치위치 | `location` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 물리적/논리적 설치 장소 | `서버실 A동 3F` |
| 229 | 공급업체 | `vendor` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 장비/시스템 공급업체명 | `Sample Vendor` |
| 230 | 모델명 | `model_name` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 장비/시스템 모델명 | `FillMaster 500` |
| 231 | 시스템 설명 | `description` | `text` | N | N | - | N | - | N | N | N | Y | 시스템 목적 및 운영 범위 설명 | `생산 공정 데이터 수집 및 제어` |
| 232 | CS 포함 여부 | `is_cs_included` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | Computerized System 포함 여부 | `TRUE` |
| 233 | 소프트웨어 버전 | `software_version` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 컴퓨터화 시스템의 소프트웨어 버전. 인벤토리 개정번호와 구분한다. | `4.8.2` |
| 234 | 인벤토리 개정번호 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 개정 기능으로 증가하는 정수 버전. 소프트웨어 버전과 별도 | `2` |
| 235 | GAMP 범주 | `gamp_category` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 화면 선택값 CATEGORY_1/CATEGORY_3/CATEGORY_4/CATEGORY_5 (GAMP 5 소프트웨어 범주: 1=인프라 소프트웨어, 3=비구성 제품, 4=구성 제품, 5=맞춤 개발). Category 2는 GAMP 5에서 폐지되어 사용하지 않는다. 미선택은 NULL | `CATEGORY_4` |
| 236 | GxP 대상 여부 | `gxp_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=대상, NOT_APPLICABLE=비대상. 미선택은 NULL | `APPLICABLE` |
| 237 | Part 11 대상 여부 | `part11_applicability` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | APPLICABLE=대상, NOT_APPLICABLE=비대상. 미선택은 NULL | `APPLICABLE` |
| 238 | 승인 상태 | `approval_status` | `varchar(30)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=작성중, REVIEW=검토중, APPROVAL=승인중, APPROVED=승인 완료, REAPPROVAL_REQUIRED=재승인 필요, REJECTED=반려, CANCELLED=취소. REJECTED·CANCELLED 후 수정은 revision_number를 증가시킨 새 개정으로 처리 | `DRAFT` |
| 239 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 240 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 241 | 사용·폐기 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE=사용, DISPOSED=폐기. 승인 상태와 별도 | `ACTIVE` |
| 242 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 전자서명 완료 시각 | `2026-09-01T00:00:00Z` |
| 243 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 화면에서 입력하는 사유 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_system_asset_1` | UNIQUE | `organization_id, management_number` | UNIQUE (organization_id, management_number) |
| `ck_system_asset_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_system_asset_3` | CHECK | `gxp_applicability` | CHECK (gxp_applicability IS NULL OR gxp_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_4` | CHECK | - | CHECK (part11_applicability IS NULL OR part11_applicability IN ('APPLICABLE','NOT_APPLICABLE')) |
| `ck_system_asset_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_system_asset_6` | CHECK | `gamp_category` | CHECK (gamp_category IS NULL OR gamp_category IN ('CATEGORY_1','CATEGORY_3','CATEGORY_4','CATEGORY_5')) |
| `ck_system_asset_7` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REAPPROVAL_REQUIRED','REJECTED','CANCELLED')) |
| `ck_system_asset_8` | CHECK | `lifecycle_status, disposed_at, disposal_reason` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0)) |
| `rule_system_asset_9` | 업무 검증 | `lifecycle_status` | DISPOSED 전환은 system_asset_revision의 DISPOSE 이벤트(폐기 전자서명 필수, rule_system_asset_revision_4)와 같은 트랜잭션으로 저장하며 disposed_at은 그 서명 시각과 같다. |

#### 업무 규칙

컴퓨터화 여부에 따라 소프트웨어 버전·GAMP·GxP·Part 11의 화면 필수값을 검사한다. 분류 3단계는 선택 가능한 조합만 허용한다.

연결 프로젝트 목록은 validation_project.system_id로 조회한다. 개정·승인·폐기 이력은 system_asset_revision 및 전자서명/감사기록으로 연결한다.

승인 완료된 정보를 변경할 때 개정번호를 증가시키고 재승인 상태를 표시한다. 소프트웨어 버전 자체의 변경과 인벤토리 문서 개정을 혼동하지 않는다.

승인되었거나 반려·취소된 업무 필드의 수정은 새 revision_number와 재승인으로 처리한다. 최신 시스템 정보의 변경은 과거 system_asset_revision 원문이나 프로젝트가 채택한 승인 개정을 바꾸지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-system_asset_revision"></a>
### 18. 시스템 인벤토리 개정 이력 (`system_asset_revision`)

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
| 244 | 인벤토리 이력 ID | `system_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 245 | 시스템 ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | system_asset.system_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 246 | 개정번호 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 처리 당시 인벤토리 개정번호 | `2` |
| 247 | 처리 구분 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | REGISTER=등록, SAVE=저장, REVISE=개정, APPROVE=승인, DISPOSE=폐기 | `REVISE` |
| 248 | 변경·처리 사유 | `change_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 또는 폐기 시 필수인 사유 | - |
| 249 | 처리 후 승인 상태 | `approval_status` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | system_asset.approval_status와 같은 코드 | `REVIEW` |
| 250 | 처리자 ID | `processed_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 251 | 처리 시각 | `processed_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이력 화면의 일시 | `2026-09-01T00:00:00Z` |
| 252 | 처리 전자서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 승인·폐기에 연결된 전자서명. 복수 서명은 동일 대상의 서명 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 253 | 원문 형식 버전 | `snapshot_schema_version` | `varchar(50)` | N | N | - | Y | `'ASSET-1'` | N | N | N | Y | asset_snapshot의 필드 구성·자료형 버전. 과거 형식 정의를 보존한다. | `ASSET-1` |
| 254 | 처리 당시 인벤토리 원문 | `asset_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 처리 후 system_asset의 모든 컬럼 값을 필드명·NULL을 포함하여 저장한다. 승인 이벤트는 최종 승인한 업무 원문과 일치해야 한다. | - |
| 255 | 원문 해시 | `snapshot_hash` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | {schema_version:snapshot_schema_version,body:asset_snapshot}을 전자서명과 같은 정규화 규칙으로 직렬화한 SHA-256 소문자 64자리. 전자서명의 전체 서명 묶음 해시와 구분한다. | - |

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
| `ck_asset_revision_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REAPPROVAL_REQUIRED','REJECTED','CANCELLED')) |

#### 업무 규칙

각 처리 이벤트와 당시 전체 원문을 함께 추가 저장한다. 이벤트 행과 원문은 수정·삭제하지 않는다. 변경 전후 비교는 audit_trail에서 조회하고, 과거 승인 원문은 APPROVE 이벤트의 asset_snapshot에서 직접 조회한다. 승인 대상은 system_asset의 system_id와 해당 revision_number로 식별한다.

APPROVE 이벤트는 검토 단계별 이벤트가 아닌 최종 승인 완료 기록이다. 서명 대상 system_id·개정번호와 승인 원문의 업무 필드가 일치해야 한다. 승인 상태·처리 시각 등 관리 메타데이터를 제외한 비교 범위는 전자서명 원문 형식에 따른다. 동일 개정의 최종 승인 원문은 한 건이며 승인 후 업무 내용 변경에는 새 개정번호를 부여한다. 저장·개정·서명·최종 승인 이벤트는 원본 변경과 같은 트랜잭션에서 기록한다.

[↑ 맨 위로](#top)

---

## Library

<a id="table-library_item"></a>
### 19. 라이브러리 항목 마스터 (`library_item`)

| 항목 | 정의 |
|---|---|
| 설명 | URS, FRA, IQ, OQ, PQ 재사용 가능 표준 라이브러리 항목 마스터 |
| Primary Key | `library_id` |
| 주요 참조(FK) | - |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 256 | 라이브러리 ID | `library_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 라이브러리 항목 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 257 | 모듈 구분 | `module_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | URS/FRA/IQ/OQ/PQ 5개 라이브러리 탭 구분 | `URS` |
| 258 | 코드 | `code` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 라이브러리 종류 내 항목 코드. (module_type, code) 복합 UNIQUE 적용 | `URS-AT-L01` |
| 259 | 분류/대분류 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | URS에서는 대분류, FRA/IQ/OQ/PQ에서는 분류를 저장한다. | `감사추적` |
| 260 | 항목명/기능명 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | URS에서는 기능명, FRA에서는 위험 항목명, IQ/OQ/PQ에서는 시험 항목명 | `데이터 변경 감사추적 자동 생성` |
| 261 | 요구사항 / 절차 | `requirement_text` | `text` | N | N | - | N | - | N | N | N | Y | URS 요구사항 본문. URS 종류에서 필수이며 다른 종류는 NULL 허용 | `모든 데이터 생성·수정·삭제 시...` |
| 262 | 기대 결과 | `expected_result` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ 예상 결과. 시험 라이브러리에서 필수 | - |
| 263 | 수용 기준/적용범위 | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | URS에서는 적용범위, IQ/OQ/PQ에서는 수용 기준. FRA는 NULL 허용 | `데이터 변경 시 Audit Trail 자동 생성...` |
| 264 | 근거 규정 | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | 화면의 규정 근거 표시·수기 입력. 구조화한 조항은 library_item_regulation에 연결하며 같은 근거를 함께 편집하지 않는다. | `21 CFR 11.10(e)` |
| 265 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 활성 여부 (TRUE/FALSE) | `TRUE` |
| 266 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 267 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 268 | 제공 유형 | `provision_type` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | URS 전용. BUILTIN=기본 제공, CUSTOM=사용자 지정 | `CUSTOM` |
| 269 | 중분류 | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 전용 중분류 | - |
| 270 | 대상 | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 전용 적용 대상 | - |
| 271 | 시험 내용 | `test_content` | `text` | N | N | - | N | - | N | N | N | Y | IQ/OQ/PQ 전용 시험 절차·내용 | - |
| 272 | 위험 시나리오 | `risk_scenario` | `text` | N | N | - | N | - | N | N | N | Y | FRA 전용 위험 시나리오 | - |
| 273 | 심각도(SEV) | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA 전용. 1~5 | - |
| 274 | 발생도(OCC) | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | FRA 전용. 1~5 | - |
| 275 | 검출도(DET) | `detectability` | `char(1)` | N | N | - | N | - | N | N | N | Y | FRA 전용. H/M/L | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_library_item_1` | UNIQUE | `module_type, code` | UNIQUE (module_type, code) |
| `ck_library_item_2` | CHECK | `module_type` | CHECK (module_type IN ('URS','FRA','IQ','OQ','PQ')) |
| `rule_library_item_3` | 업무 검증 | - | URS: 제공 유형·대분류·중분류·대상·기능명·요구사항·적용범위 필수. FRA: 분류·위험 항목명·시나리오·SEV·OCC·DET 필수. IQ/OQ/PQ: 분류·항목명·시험 내용·예상 결과·수용 기준 필수. |
| `rule_library_item_4` | 업무 검증 | `severity` | FRA의 severity/occurrence는 1~5, detectability는 H/M/L 중 하나이다. |
| `ck_library_item_provision_type` | CHECK | `provision_type` | CHECK (provision_type IS NULL OR provision_type IN ('BUILTIN','CUSTOM')) |

#### 업무 규칙

URS의 대분류는 category, 기능명은 title, 적용범위는 acceptance_criteria에서 관리한다.

템플릿을 요구사항·위험·시험에 적용할 때 본문과 입력값을 복사한다. 이후 라이브러리 수정이 이미 작성·승인한 항목을 자동 변경하지 않는다.

[↑ 맨 위로](#top)

---

## Validation

<a id="table-validation_project"></a>
### 20. Validation 프로젝트 (`validation_project`)

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
| 276 | 프로젝트 ID | `project_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 277 | 프로젝트 코드 | `project_code` | `varchar(100)` | N | N | - | Y | - | Y | Y | N | Y | 프로젝트 식별 코드 | `VP-SYS-008-20260422` |
| 278 | 프로젝트명 | `project_name` | `varchar(200)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트명 | `테스트 장비3 CSV 프로젝트` |
| 279 | 시스템 ID | `system_id` | `uuid` | N | Y | `system_asset.system_id` | Y | - | N | Y | N | Y | system_asset.system_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 280 | 상태 | `status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS=진행중, CLOSED_NORMAL=정상종료, CLOSED_FORCED=강제종료 | `IN_PROGRESS` |
| 281 | 검증 방식 | `validation_type` | `varchar(50)` | N | N | - | Y | `'NEW'` | N | N | N | Y | NEW=신규, CHANGE=변경, REVALIDATION=재검증 | `NEW` |
| 282 | 밸리데이션 레벨 | `validation_level` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | LEVEL_1/LEVEL_2/LEVEL_3/CUSTOM. CUSTOM=사용자 지정이며 수행 활동은 project_activity로 저장 | `LEVEL_2` |
| 283 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 추가 메모 사항 | - |
| 284 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-25T00:00:00Z` |
| 285 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 생성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 286 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 287 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 288 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 289 | 종료 유형 | `closure_type` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 유형 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `FORCED` |
| 290 | 강제종료 사유 | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | 현재/최종 project_closure_request의 강제종료 사유 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `사업 우선순위 변경으로 Validation 중단` |
| 291 | 종료 요청자 ID | `closure_requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 요청자 ID 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 292 | 종료 요청 시각 | `closure_requested_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 요청 시각 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `2026-09-14T09:00:00Z` |
| 293 | 종료 완료 시각 | `closed_at` | `timestamptz` | N | N | - | N | - | N | Y | N | Y | 현재/최종 project_closure_request의 종료 완료 시각 요약. 요청 원본에서 동기화하며 독립 편집하지 않는다. | `2026-09-14T09:05:00Z` |
| 294 | 현재 종료 요청 ID | `current_closure_request_id` | `uuid` | N | Y | `project_closure_request.closure_request_id` | N | - | N | Y | N | Y | 종료 진행 상태 및 사유를 표시할 종료 요청 | `00000000-0000-0000-0000-000000000001` |
| 295 | 현재 검증 대상 기준 | `current_system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 현재 is_current=TRUE인 동일 프로젝트의 기준. 최초 업무 상신 또는 시험 수행 전에 필수 | - |
| 296 | 적용 선후행 조건 버전 | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 생성 시 채택한 선후행 규칙 묶음 버전(activity_dependency.dependency_set_version) | `DEPENDENCY-1` |
| 297 | 적용 선후행 조건 원문 | `dependency_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 생성 당시 활성 조건 전체와 판정 의미를 보존. {schema_version,version,definitions,conditions:[{activity_dependency_id,successor_activity_id,predecessor_activity_id,dependency_type,required_status,condition_type,condition_value,evaluation_order}]} | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_validation_project_1` | CHECK | `status` | CHECK (status IN ('IN_PROGRESS','CLOSED_NORMAL','CLOSED_FORCED')) |
| `ck_validation_project_2` | CHECK | `validation_type` | CHECK (validation_type IN ('NEW','CHANGE','REVALIDATION')) |
| `ck_validation_project_3` | CHECK | `validation_level` | CHECK (validation_level IN ('LEVEL_1','LEVEL_2','LEVEL_3','CUSTOM')) |
| `rule_project_current_refs` | 업무 검증 | `current_closure_request_id, current_system_baseline_id` | 현재 종료 요청과 검증 대상 기준은 모두 이 프로젝트에 속해야 한다. 기준이 존재하면 현재 기준 참조는 is_current=TRUE인 유일한 행과 일치해야 한다. |
| `ck_validation_project_closure_type` | CHECK | `closure_type` | CHECK (closure_type IS NULL OR closure_type IN ('NORMAL','FORCED')) |

#### 업무 규칙

화면의 프로젝트명·대상 시스템·검증 유형·수준·메모는 이 테이블에서 관리한다. 진행률은 선택 수행 활동의 완료 상태로 계산하고 RTM은 분모에서 제외한다.

프로젝트의 검증 대상 GAMP·GxP·소프트웨어 버전은 current_system_baseline_id가 가리키는 승인 원문에서 조회한다. 시스템 최신 정보와 검증 대상 기준을 구분하여 표시한다.

종료 요청의 검토/승인 진행은 project_closure_request 및 workflow_instance에 저장한다. 최종 승인 후에만 프로젝트를 정상종료 또는 강제종료로 변경한다.

프로젝트 생성 시 dependency_set_version과 dependency_snapshot을 함께 고정하며, 전역 activity_dependency 변경을 진행 중·종료 프로젝트에 자동 전파하지 않는다. 조건의 정의에는 상태별 완료 판정, AND/OR 결합, 미선택 활동 처리 및 위험 커버리지 판정 기준을 포함한다. 프로젝트 평가에는 이 사본을 사용하고 생성 후 직접 교체하지 않는다. 업무 승인·시험 수행 전에 유효한 current_system_baseline_id를 지정하며, 최초 기준 등록과 현재 참조 설정은 한 트랜잭션으로 처리한다.

[↑ 맨 위로](#top)

---

<a id="table-project_system_baseline"></a>
### 21. 프로젝트 검증 대상 개정 (`project_system_baseline`)

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
| 298 | 검증 대상 기준 ID | `baseline_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트가 채택한 한 번의 검증 대상 기준 식별자 | - |
| 299 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 검증 대상 기준이 속하는 프로젝트 | - |
| 300 | 인벤토리 승인 이력 ID | `system_revision_id` | `uuid` | N | Y | `system_asset_revision.system_revision_id` | Y | - | N | Y | N | Y | action_type=APPROVE인 변경 불가능한 원문 이벤트 | - |
| 301 | 기준 순번 | `baseline_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 프로젝트별 1부터 증가하며 변경 시 새 행을 생성한다. | `1` |
| 302 | 채택·변경 사유 | `change_reason` | `text` | N | N | - | Y | - | N | N | N | Y | 최초 대상 선정 또는 승인 개정 변경의 사유. 공백 불가 | - |
| 303 | 현재 적용 여부 | `is_current` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 현재 프로젝트 기준 여부. 과거 행은 FALSE로 변경하고 원문 연결은 유지한다. | - |
| 304 | 채택 시각 | `adopted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 서버에서 기록한 적용 시각 | - |
| 305 | 채택자 | `adopted_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 대상 설정 권한이 있는 처리자 | - |

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
### 22. 프로젝트 참여자 (`project_member`)

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
| 306 | 프로젝트 참여자 ID | `project_member_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 참여자 역할 매핑 고유 식별자. deleted_at IS NULL이고 member_status가 ACTIVE인 행에 (project_id, user_id, role_id) 중복을 허용하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 307 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 참여자가 소속된 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 308 | 사용자 ID | `user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트에 참여하는 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 309 | 역할 ID | `role_id` | `uuid` | N | Y | `role.role_id` | Y | - | N | Y | N | Y | 프로젝트 내에서 사용자가 수행하는 역할 ID | `00000000-0000-0000-0000-000000000001` |
| 310 | 참여 상태 | `member_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | Y | N | Y | 프로젝트 참여 상태. ACTIVE, INACTIVE, WITHDRAWN | `ACTIVE` |
| 311 | 참여 시작 시각 | `joined_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여가 시작된 시각 | `2026-09-01T10:00:00Z` |
| 312 | 참여 종료 시각 | `left_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여가 종료된 시각. 현재 참여 중이면 NULL | `2026-09-02T10:00:00Z` |
| 313 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 314 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 등록한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 315 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 참여 정보 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 316 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 참여 정보를 최종 수정한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 317 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 참여 정보 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_member_1` | UNIQUE | `project_id, user_id, role_id, deleted_at, member_status` | UNIQUE (project_id, user_id, role_id) WHERE deleted_at IS NULL AND member_status='ACTIVE' |
| `ck_project_member_status` | CHECK | `member_status` | CHECK (member_status IN ('ACTIVE','INACTIVE','WITHDRAWN')) |

#### 업무 규칙

프로젝트별 참여자와 역할을 관리한다. 화면의 조회/편집/폐기 권한은 access_permission_grant에서 별도로 관리한다.

[↑ 맨 위로](#top)

---

<a id="table-validation_activity"></a>
### 23. 밸리데이션 활동 마스터 (`validation_activity`)

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
| 318 | 활동 ID | `activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 밸리데이션 활동 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 319 | 활동 코드 | `activity_code` | `varchar(50)` | N | N | - | Y | - | Y | Y | N | Y | VP, VA, QIA, URS, FDS_GROUP, FRA, DQ, IQ, OQ, PQ, VSR. FDS_GROUP은 화면의 F&DS 표시명에 대응 | `FDS_GROUP` |
| 320 | 활동명 | `activity_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 화면 표시용 활동명 | `기능 및 상세 설계 명세` |
| 321 | 활동 순서 | `display_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 기본 표시 순서: VP→VA→QIA→URS→F&DS→FRA→DQ→IQ→OQ→PQ→VSR. F&DS 내부 문서는 FDS 다음 DDS | `5` |
| 322 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 활동 마스터 사용 여부 | `TRUE` |
| 323 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 324 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 등록한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 325 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 활동 마스터 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 326 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 활동 마스터를 최종 수정한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 327 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 마스터 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_validation_activity_code` | CHECK | `activity_code` | CHECK (activity_code IN ('VP','VA','QIA','URS','FDS_GROUP','FRA','DQ','IQ','OQ','PQ','VSR')) |

#### 업무 규칙

RTM은 요구사항·설계·위험·시험의 추적 관계를 조회하는 대시보드 기능이다. 시스템 인벤토리는 프로젝트의 선행 기준 정보로 관리한다.

FDS와 DDS는 F&DS 한 활동에 속하는 문서 종류다. 두 종류를 각각 별도의 수행 단계로 중복 생성하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-project_activity"></a>
### 24. 프로젝트 수행 활동 (`project_activity`)

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
| 328 | 프로젝트 활동 ID | `project_activity_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 프로젝트 수행 활동 고유 식별자. deleted_at IS NULL인 행에 (project_id, activity_id) 중복을 허용하지 않는다. 수행 대상 선택 여부와 무관하게 적용한다. | `00000000-0000-0000-0000-000000000001` |
| 329 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 활동이 속한 Validation 프로젝트 | `00000000-0000-0000-0000-000000000001` |
| 330 | 활동 ID | `activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 프로젝트에서 수행할 활동 | `00000000-0000-0000-0000-000000000001` |
| 331 | 수행 대상 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 프로젝트 설정에서 선택한 실제 수행 활동인지 여부. RTM 선택행을 만들지 않는다. | `TRUE` |
| 332 | 필수 활동 여부 | `is_required` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 해당 프로젝트에서 생략할 수 없는 활동인지 여부 | `TRUE` |
| 333 | 활동 상태 | `activity_status` | `varchar(20)` | N | N | - | Y | `'LOCKED'` | N | Y | N | Y | 활동 상태. LOCKED, READY, IN_PROGRESS, COMPLETED, APPROVED, SKIPPED. 현재 Revision 기준으로 집계하며 산출물 변경·개정 시 재평가한다. COMPLETED/APPROVED는 단순 화면 입력값으로 변경하지 않는다 | `READY` |
| 334 | 활성화 시각 | `activated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 선행 조건 충족으로 활동이 READY가 된 시각 | `2026-09-01T10:00:00Z` |
| 335 | 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 시작 시각 | `2026-09-01T11:00:00Z` |
| 336 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동 수행 완료 시각 | `2026-09-02T15:00:00Z` |
| 337 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 활동의 최종 승인 완료 시각 | `2026-09-02T17:00:00Z` |
| 338 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 339 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 등록 사용자 | `00000000-0000-0000-0000-000000000001` |
| 340 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 프로젝트 활동 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 341 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 프로젝트 활동 최종 수정 사용자 | `00000000-0000-0000-0000-000000000001` |
| 342 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 프로젝트 활동 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_activity_1` | UNIQUE | `project_id, activity_id, deleted_at` | UNIQUE (project_id, activity_id) WHERE deleted_at IS NULL |
| `ck_project_activity_status` | CHECK | `activity_status` | CHECK (activity_status IN ('LOCKED','READY','IN_PROGRESS','COMPLETED','APPROVED','SKIPPED')) |

#### 업무 규칙

진행률·완료 활동 수·정상종료 조건은 선택된 실제 수행 활동을 대상으로 한다. RTM 자체 승인·산출물은 요구하지 않는다. FDS/DDS의 진행은 하나의 F&DS 활동에 집계한다.

[↑ 맨 위로](#top)

---

<a id="table-activity_dependency"></a>
### 25. 활동 선후행 조건 (`activity_dependency`)

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
| 343 | 활동 선후행 조건 ID | `activity_dependency_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 활동 선후행 조건 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 344 | 조건 묶음 버전 | `dependency_set_version` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 이 조건이 속한 선후행 조건 묶음의 버전. 같은 버전의 행 전체가 하나의 묶음이며, 프로젝트가 채택한 버전의 행은 수정·비활성화·삭제하지 않는다. 변경은 새 버전으로 전체 조건 행을 생성한다. validation_project.dependency_set_version이 이 값을 가리킨다 | `DEPENDENCY-1` |
| 345 | 후행 활동 ID | `successor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | Y | - | N | Y | N | Y | 조건 충족 후 활성화되는 활동 | `00000000-0000-0000-0000-000000000001` |
| 346 | 선행 활동 ID | `predecessor_activity_id` | `uuid` | N | Y | `validation_activity.activity_id` | N | - | N | Y | N | Y | 후행 활동 활성화 전에 확인할 활동. 전체 활동 조건이면 NULL 허용. STATUS 조건이면 필수이다 | `00000000-0000-0000-0000-000000000001` |
| 347 | 관계 구분 | `dependency_type` | `varchar(20)` | N | N | - | Y | `'REQUIRED'` | N | Y | N | Y | 선후행 관계 구분. REQUIRED, RECOMMENDED | `REQUIRED` |
| 348 | 요구 상태 | `required_status` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | STATUS 조건의 판정 기준. CREATED는 유효한 현재 산출물 존재, COMPLETED는 수행 완료, APPROVED는 현재 산출물 최종 승인을 뜻한다. project_activity.activity_status에 CREATED를 저장하지 않는다. STATUS 조건에서만 필수이며 상세 판정은 아래 업무 규칙을 따른다. | `APPROVED` |
| 349 | 조건 유형 | `condition_type` | `varchar(50)` | N | N | - | Y | `'STATUS'` | N | Y | N | Y | STATUS, ACTIVITY_SELECTED, TRACEABILITY_EXISTS, HIGH_RISK_COVERED, OPEN_DEVIATION_ZERO, ALL_SELECTED_APPROVED. 화면의 선행 단계 및 완료 조건을 판정하는 구분 | `STATUS` |
| 350 | 조건 값 | `condition_value` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 조건 판정 인자. STATUS + CREATED에서는 허용된 대상 산출물 테이블명(예: requirement)을 저장한다. 다른 조건의 인자는 해당 조건 구현에서 검증하며 불필요하면 NULL로 둔다. | - |
| 351 | 조건 설명 | `condition_description` | `text` | N | N | - | Y | - | N | N | N | Y | 활동 활성화에 필요한 선행 활동과 승인 상태의 설명 | `URS 승인완료` |
| 352 | 평가 순서 | `evaluation_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 동일 후행 활동의 조건 평가 순서. 평가 순서는 조건 간 AND/OR 관계를 바꾸지 않는다 | `1` |
| 353 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 활성화 조건 사용 여부 | `TRUE` |
| 354 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 355 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 등록 사용자 | `00000000-0000-0000-0000-000000000001` |
| 356 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 조건 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 357 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조건 최종 수정 사용자 | `00000000-0000-0000-0000-000000000001` |
| 358 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 조건 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_activity_dependency_type` | CHECK | `dependency_type` | CHECK (dependency_type IN ('REQUIRED','RECOMMENDED')) |
| `ck_activity_dependency_required_status` | CHECK | `required_status` | CHECK (required_status IS NULL OR required_status IN ('CREATED','COMPLETED','APPROVED')) |
| `ck_activity_dependency_condition_type` | CHECK | `condition_type` | CHECK (condition_type IN ('STATUS','ACTIVITY_SELECTED','TRACEABILITY_EXISTS','HIGH_RISK_COVERED','OPEN_DEVIATION_ZERO','ALL_SELECTED_APPROVED')) |
| `ck_activity_dependency_status_condition` | CHECK | `condition_type, predecessor_activity_id, required_status` | CHECK ((condition_type='STATUS' AND predecessor_activity_id IS NOT NULL AND required_status IS NOT NULL) OR (condition_type<>'STATUS' AND required_status IS NULL)) |
| `ck_activity_dependency_not_self` | CHECK | `successor_activity_id, predecessor_activity_id` | CHECK (predecessor_activity_id IS NULL OR predecessor_activity_id <> successor_activity_id) |
| `uq_activity_dependency_condition` | UNIQUE | `dependency_set_version, successor_activity_id, predecessor_activity_id, condition_type, condition_value, is_active, deleted_at` | UNIQUE (dependency_set_version, successor_activity_id, COALESCE(predecessor_activity_id, '00000000-0000-0000-0000-000000000000'::uuid), condition_type, COALESCE(condition_value, '')) WHERE is_active=TRUE AND deleted_at IS NULL |
| `rule_activity_dependency_cycle` | 업무 검증 | `successor_activity_id, predecessor_activity_id` | 같은 dependency_set_version의 활성 조건으로 선후행 순환이 생기면 저장하지 않는다. |
| `rule_activity_dependency_version_lock` | 업무 검증 | `dependency_set_version` | validation_project.dependency_set_version으로 채택된 버전의 행은 수정·비활성화·삭제하지 않는다. |

#### 업무 규칙

전체 활동 승인 조건은 선택된 실제 수행 활동만 평가한다. RTM은 선행·후행 활동 또는 독립 승인 조건으로 등록하지 않는다.

STATUS 조건에서는 predecessor_activity_id와 required_status가 필수이며 required_status는 CREATED/COMPLETED/APPROVED이다. 다른 조건에서는 필요하지 않은 선행 활동·상태 값은 NULL로 둔다. 활성 선후행 관계에 순환을 허용하지 않는다.

이 테이블은 신규 프로젝트에 적용할 전역 조건 템플릿이다. 변경 시 새 dependency_set_version으로 전체 조건 행을 생성하여 발행하며, 프로젝트가 채택한 버전의 행은 수정·비활성화·삭제하지 않는다. 조건 유형별 판정 의미(definitions)는 버전 이름으로 식별하는 고정 정의이며 같은 버전 이름에 다른 정의를 등록하지 않는다(QIA·FRA 판정 규칙 버전과 같은 방식). 프로젝트 생성 시 선택한 버전의 전체 내용을 validation_project.dependency_snapshot에 복사하며, 이후 조건의 수정·비활성화·삭제는 기존 프로젝트의 조건을 변경하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-project_closure_request"></a>
### 26. 프로젝트 종료 요청 (`project_closure_request`)

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
| 359 | 종료 요청 ID | `closure_request_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 360 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 361 | 요청 순번 | `request_version` | `integer` | N | N | - | Y | - | N | N | N | Y | 같은 프로젝트의 종료 재요청 구분 | `1` |
| 362 | 종료 유형 | `closure_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | NORMAL=정상종료, FORCED=강제종료 | `NORMAL` |
| 363 | 요청 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT=작성중, REVIEW=검토중, APPROVAL=승인중, APPROVED=완료, REJECTED=반려, CANCELLED=취소 | `REVIEW` |
| 364 | 종료 사유 | `reason` | `text` | N | N | - | N | - | N | N | N | Y | 강제종료일 때 필수 | - |
| 365 | 요청 시 진행률 | `progress_snapshot` | `numeric(5,2)` | N | N | - | Y | - | N | N | N | Y | 종료 확인 화면에 표시한 요청 당시 진행률. RTM 제외 | `100.00` |
| 366 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 367 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 요청 시각 값 | `2026-09-01T00:00:00Z` |
| 368 | 종료 결재 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 369 | 종료 처리 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 종료 처리 시각 값 | `2026-09-01T00:00:00Z` |
| 370 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 371 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 372 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 373 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_project_closure_request_1` | UNIQUE | `project_id, request_version` | UNIQUE (project_id, request_version) |
| `ck_project_closure_request_2` | CHECK | `request_version` | CHECK (request_version >= 1) |
| `ck_project_closure_request_3` | CHECK | `progress_snapshot` | CHECK (progress_snapshot BETWEEN 0 AND 100) |
| `rule_project_closure_request_4` | 업무 검증 | `closure_type` | closure_type='FORCED'이면 reason은 비어 있을 수 없다. |
| `rule_project_closure_request_5` | 업무 검증 | - | 동일 프로젝트의 DRAFT/REVIEW/APPROVAL 상태 종료 요청은 최대 한 건이다. |
| `uq_project_closure_active` | UNIQUE | `project_id` | UNIQUE (project_id) WHERE status IN ('DRAFT','REVIEW','APPROVAL') |
| `ck_project_closure_request_status` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_project_closure_request_type` | CHECK | `closure_type` | CHECK (closure_type IN ('NORMAL','FORCED')) |

#### 업무 규칙

정상종료는 선택된 수행 활동과 필요한 산출물의 완료·승인 상태를 확인한다. RTM 독립 승인과 RTM 산출물은 종료 조건에서 제외한다.

서명자는 전자서명 및 결재 처리 이력으로 연결한다. workflow_instance의 대상은 project_closure_request/closure_request_id/CLOSE-{request_version}이고 프로젝트가 일치해야 한다. 최종 승인 후 validation_project.status와 종료 요약을 함께 반영하며 검토중/승인중은 프로젝트 종료 상태로 처리하지 않는다.

[↑ 맨 위로](#top)

---

## VP

<a id="table-vp_plan"></a>
### 27. 검증 계획 (`vp_plan`)

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
| 374 | VP 개정 ID | `vp_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 375 | 개정 간 논리 ID | `vp_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 376 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 377 | 계획 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 검증 계획의 표시 제목 | `시스템 검증 계획` |
| 378 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 379 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 380 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 381 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 382 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 383 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 384 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 385 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 386 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 387 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 388 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 389 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 390 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 391 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vp_plan_1` | UNIQUE | `vp_key, revision_number` | UNIQUE (vp_key, revision_number) |
| `ck_vp_plan_1_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vp_plan_2` | UNIQUE | `vp_key, is_current_version` | UNIQUE (vp_key) WHERE is_current_version = TRUE |
| `ck_vp_plan_3` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_vp_plan_4` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vp_plan_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `rule_vp_plan_5` | 업무 검증 | `workflow_instance_id` | 프로젝트당 VP 논리키는 하나이며 개정은 여러 건이다. APPROVED에는 workflow_instance_id 필수. |
| `ck_vp_plan_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

목차 추가·삭제·정렬·포함 여부·본문을 vp_section에 저장한다. 생성 산출물의 편집 내용과 문서 승인 상태는 deliverable_revision에서 관리하며 원본 VP와 구분한다.

같은 vp_key의 모든 개정은 동일 프로젝트에 속한다. 개정 생성과 최신 개정 전환은 프로젝트 및 논리 항목을 잠그고 같은 트랜잭션에서 처리한다.

[↑ 맨 위로](#top)

---

<a id="table-vp_section"></a>
### 28. 검증 계획 목차 (`vp_section`)

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
| 392 | 목차 ID | `vp_section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 393 | VP 개정 ID | `vp_id` | `uuid` | N | Y | `vp_plan.vp_id` | Y | - | N | Y | N | Y | vp_plan.vp_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 394 | 목차 논리 키 | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 개정 간 유지하는 목차 식별 키 | `purpose` |
| 395 | 목차 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 목차 제목 | `목적` |
| 396 | 목차 내용 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | 목차 내용 | `본 계획의 검증 목적` |
| 397 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 398 | 산출물 포함 여부 | `is_enabled` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 산출물 포함 여부 | `TRUE` |
| 399 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 400 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 401 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 402 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vp_section_1` | UNIQUE | `vp_id, section_key` | UNIQUE (vp_id, section_key) |
| `uq_vp_section_1_2` | UNIQUE | `vp_id, sort_order` | UNIQUE (vp_id, sort_order) |
| `ck_vp_section_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |

#### 업무 규칙

상신(REVIEW) 이후 VP의 하위 목차는 직접 수정하지 않으며, 수정이 필요하면 새 VP 개정에 복사하여 편집한다. AI 초안 생성 후 사용자가 저장한 내용도 content에 기록한다.

[↑ 맨 위로](#top)

---

## QIA

<a id="table-qia_assessment"></a>
### 29. 품질 영향 평가 헤더 (`qia_assessment`)

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
| 403 | QIA ID | `qia_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 평가 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 404 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 405 | Part 11 Q1 전자기록의 종이기록 대체 여부 | `p11_q1` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 전자 기록이 종이 기록을 대체합니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 406 | Part 11 Q2 전자서명 사용 여부 | `p11_q2` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 전자 서명을 사용합니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 407 | Part 11 Q3 기록 이력의 규제 증빙 여부 | `p11_q3` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 기록 생성·변경 이력이 규제 증빙입니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 408 | Part 11 Q4 접근 통제·사용자 식별 필요 여부 | `p11_q4` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 접근 통제와 사용자 식별이 필요합니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 409 | Part 11 Q5 장기 보관·검색 필요 여부 | `p11_q5` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 기록의 장기 보관과 검색이 필요합니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 410 | Part 11 Q6 시스템 간 전자기록 전송 여부 | `p11_q6` | `varchar(3)` | N | N | - | N | - | N | N | N | Y | 시스템 간 전자 기록 전송이 있습니까? 응답 Yes / No. 작성 중 미응답은 NULL이며 QIA 문서 상신 시 필수 | `No` |
| 411 | Part11 평가 결론 | `part11_result` | `text` | N | N | - | N | - | N | N | N | N | 6개 응답에서 자동 계산한 표시값. 미응답 문항이 있으면 NULL(미평가), 모두 응답한 경우 하나라도 Yes이면 적용, 모두 No이면 비적용. 직접 입력하지 않음 | `Part 11 비적용` |
| 412 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 413 | 질문 세트 버전 | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | 6개 질문 문구·순서를 고정하는 버전 | `UI-P11-1` |
| 414 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-P11-1'` | N | N | N | Y | 응답 중 하나 이상이 Yes이면 적용으로 판정하는 규칙의 버전 | `UI-P11-1` |
| 415 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 416 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 417 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 418 | 적용 질문 원문 | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. 선택한 버전의 모든 질문·문구·순서·응답 선택값을 저장한다. | - |
| 419 | 적용 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. 논리식·응답 해석·판정값 매핑을 포함한 전체 정의 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_qia_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |
| `rule_qia_assessment_2` | 업무 검증 | `part11_result` | part11_result는 응답과 같은 트랜잭션에서 재계산하는 캐시이며 독립 수정 금지. 미응답 문항이 있으면 NULL(미평가)이다. |
| `ck_qia_assessment_3` | CHECK | `p11_q1, p11_q2, p11_q3, p11_q4, p11_q5, p11_q6` | CHECK ((p11_q1 IS NULL OR p11_q1 IN ('Yes','No')) AND (p11_q2 IS NULL OR p11_q2 IN ('Yes','No')) AND (p11_q3 IS NULL OR p11_q3 IN ('Yes','No')) AND (p11_q4 IS NULL OR p11_q4 IN ('Yes','No')) AND (p11_q5 IS NULL OR p11_q5 IN ('Yes','No')) AND (p11_q6 IS NULL OR p11_q6 IN ('Yes','No'))) |
| `rule_qia_assessment_4` | 업무 검증 | `p11_q1, p11_q2, p11_q3, p11_q4, p11_q5, p11_q6` | QIA 문서(deliverable_revision) 상신 시 p11_q1~p11_q6이 응답되어 있고, 문서에 포함된 모든 모듈은 프로세스가 1건 이상이며 모든 프로세스의 q1_val~q10_val가 응답되어 있어야 한다. 미응답 문항이 있으면 상신을 거부한다. |

#### 업무 규칙

Part 11 질문은 모듈별이 아닌 QIA 공통 응답이다. QIA 문서 생성·승인 시 질문 버전·응답·규칙 버전·결과를 deliverable_revision의 원본/표 스냅샷에 고정한다. 현재 응답 변경으로 과거 승인 문서가 바뀌지 않는다.

문서의 승인 상태와 버전은 deliverable_document 및 deliverable_revision에서 관리한다. 모듈별 GxP 판정은 하위 프로세스에서 조회한다.

평가 작성 시 버전별 질문·규칙 정의를 복사하고 version 값이 question_set_version 및 rule_version과 일치하는지 서버에서 검증한다. 동일 버전 이름에 다른 정의를 등록하지 않는다. 새 전역 버전을 기존 평가에 자동 적용하지 않는다. 계산에는 저장한 정의를 사용하며 단순한 함수명이나 버전 문자열만으로 원문을 대체하지 않는다.

Part 11 공통 응답 또는 적용 정의를 변경해도 이미 상신·승인한 문서의 evaluation_definition_snapshot은 변경하지 않는다. QIA 문서 상신 시 공통 평가와 포함된 모든 프로세스의 전체 질문·규칙·응답·판정을 evaluation_definition_snapshot에 함께 고정한다.

[↑ 맨 위로](#top)

---

<a id="table-qia_module_item"></a>
### 30. QIA 모듈 상세 평가 (`qia_module_item`)

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
| 420 | QIA 모듈 항목 ID | `qia_module_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | QIA 모듈 평가 항목 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 421 | 개정 간 논리 ID | `module_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 422 | QIA ID | `qia_id` | `uuid` | N | Y | `qia_assessment.qia_id` | Y | - | N | Y | N | Y | 상위 QIA 평가 식별자 | `00000000-0000-0000-0000-000000000001` |
| 423 | 모듈 코드 | `module_code` | `varchar(20)` | N | N | - | N | - | N | Y | N | Y | 선택 내부 모듈 코드. 화면 승인번호 item_number와 구분 | `MOD-001` |
| 424 | 모듈명 | `module_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 상위 모듈명 | `품질관리` |
| 425 | 모듈 설명 | `module_description` | `text` | N | N | - | N | - | N | N | N | Y | 모듈의 범위 및 목적 설명 | `품질관리 관련 종합 평가` |
| 426 | 평가 결과 | `result_type` | `varchar(20)` | N | N | - | N | - | N | N | N | N | 프로세스 판정의 집계 캐시. 프로세스가 없거나 미응답 문항이 있는 프로세스가 있으면 NULL(미평가), 그 외 한 프로세스라도 GXP이면 GXP, 아니면 NON_GXP. 직접 편집하지 않음 | `GXP` |
| 427 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-26T10:00:00Z` |
| 428 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-26T10:00:00Z` |
| 429 | 모듈 표시 순서 | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 모듈 표시 순서 | `1` |
| 430 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 431 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 432 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 433 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 434 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 435 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 436 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `QIA-001` |
| 437 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 438 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 439 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 440 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 441 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 442 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_qia_module_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_qia_module_item_2` | UNIQUE | `module_key, revision_number` | UNIQUE (module_key, revision_number) |
| `ck_qia_module_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_qia_module_item_3` | UNIQUE | `module_key, is_current_version` | UNIQUE (module_key) WHERE is_current_version = TRUE |
| `ck_qia_module_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_qia_module_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_qia_module_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_qia_module_item_6` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_module_item_6` | 업무 검증 | `sort_order` | 모듈 먼저 등록 후 프로세스를 추가할 수 있으므로 하위 프로세스 0건 초안을 허용한다. 상신(REVIEW)에는 하위 프로세스가 1건 이상 있어야 한다 |
| `rule_qia_module_item_7` | 업무 검증 | `status` | 모듈 개정 상신(REVIEW) 시 하위 프로세스가 1건 이상 있고 모든 프로세스의 q1_val~q10_val가 응답되어 있어야 한다. 미응답 문항이 있으면 상신을 거부한다. |
| `ck_qia_module_item_result_type` | CHECK | `result_type` | CHECK (result_type IS NULL OR result_type IN ('GXP','NON_GXP')) |
| `ck_qia_module_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

프로세스가 없는 모듈은 판정하지 않으며(result_type NULL, 미평가) 상신할 수 없다. 평가하지 않은 모듈이 NON_GXP로 승인되지 않도록 한다.

모듈과 프로세스는 1:N 관계이며 프로세스명과 10개 질문 응답은 qia_process에 저장한다. 개정 시 프로세스도 새 모듈 개정 아래 복제하며 승인 모듈의 프로세스 원문·응답은 불변이다.

같은 module_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-qia_process"></a>
### 31. QIA 프로세스 평가 (`qia_process`)

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
| 443 | 프로세스 ID | `qia_process_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 444 | 모듈 개정 ID | `qia_module_item_id` | `uuid` | N | Y | `qia_module_item.qia_module_item_id` | Y | - | N | Y | N | Y | qia_module_item.qia_module_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 445 | 프로세스 논리 ID | `process_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | N | N | Y | 모듈 개정 간 유지하는 프로세스 식별자 | `00000000-0000-0000-0000-000000000001` |
| 446 | 내부 프로세스 코드 | `process_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 내부 프로세스 코드 | - |
| 447 | 프로세스명 | `process_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 프로세스명 | `시험 결과 입력` |
| 448 | 프로세스 설명 | `process_description` | `text` | N | N | - | N | - | N | N | N | Y | 프로세스 설명 | - |
| 449 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 표시 순서 | `1` |
| 450 | 질문 세트 버전 | `question_set_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | 질문 세트 버전 | `UI-GXP-1` |
| 451 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-GXP-1'` | N | N | N | Y | 판정 규칙 버전 | `UI-GXP-1` |
| 452 | GxP Q1 | `q1_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 제품 품질, 환자 안전 또는 데이터 무결성에 직접 영향을 주는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 453 | GxP Q2 | `q2_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | GxP 의사결정에 사용되는 데이터를 생성하거나 처리하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 454 | GxP Q3 | `q3_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 배치 출하 또는 품질 승인 절차를 지원하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 455 | GxP Q4 | `q4_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 규제기관 제출 또는 검사 증빙에 사용되는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 456 | GxP Q5 | `q5_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 전자 기록의 생성·수정·보관을 수행하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 457 | GxP Q6 | `q6_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 장비 또는 공정의 중요 파라미터를 제어하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 458 | GxP Q7 | `q7_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 일탈, CAPA, 변경 관리 프로세스를 지원하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 459 | GxP Q8 | `q8_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 사용자 권한이 품질 관련 기능 접근을 통제하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 460 | GxP Q9 | `q9_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 외부 GxP 시스템과 중요 데이터를 교환하는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 461 | GxP Q10 | `q10_val` | `varchar(5)` | N | N | - | N | - | N | N | N | Y | 백업·복구 실패가 GxP 기록에 영향을 주는가? 응답 O / X / ▲. 작성 중 미응답은 NULL이며 모듈 상신 시 필수 | `X` |
| 462 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 463 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 464 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 465 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 466 | 적용 질문 원문 | `question_set_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,questions:[{question_id,field_name,text,sort_order,answer_options}]}. 선택한 버전의 모든 질문·문구·순서·응답 선택값을 저장한다. | - |
| 467 | 적용 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_fields,normalization,expression,result_mapping}. 논리식·응답 해석·판정값 매핑을 포함한 전체 정의 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_qia_process_1` | UNIQUE | `qia_module_item_id, process_key` | UNIQUE (qia_module_item_id, process_key) |
| `uq_qia_process_1_2` | UNIQUE | `qia_module_item_id, sort_order` | UNIQUE (qia_module_item_id, sort_order) |
| `ck_qia_process_1_3` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_qia_process_2` | 업무 검증 | `q1_val` | q1_val~q10_val는 작성 중 NULL(미응답)을 허용하며 모듈 상신 시 모두 응답해야 한다(rule_qia_module_item_7). |
| `ck_qia_process_3` | CHECK | `q1_val, q2_val, q3_val, q4_val, q5_val, q6_val, q7_val, q8_val, q9_val, q10_val` | CHECK ((q1_val IS NULL OR q1_val IN ('O','X','▲')) AND (q2_val IS NULL OR q2_val IN ('O','X','▲')) AND (q3_val IS NULL OR q3_val IN ('O','X','▲')) AND (q4_val IS NULL OR q4_val IN ('O','X','▲')) AND (q5_val IS NULL OR q5_val IN ('O','X','▲')) AND (q6_val IS NULL OR q6_val IN ('O','X','▲')) AND (q7_val IS NULL OR q7_val IN ('O','X','▲')) AND (q8_val IS NULL OR q8_val IN ('O','X','▲')) AND (q9_val IS NULL OR q9_val IN ('O','X','▲')) AND (q10_val IS NULL OR q10_val IN ('O','X','▲'))) |

#### 업무 규칙

GxP 판정 규칙: Q1=O이면서 Q2~Q10 중 O가 하나 이상이면 GXP, 그 외 NON_GXP. ▲는 O 조건에 포함하지 않는다. 판정은 응답에서 조회 계산한다. 미응답(NULL) 문항이 있으면 판정하지 않고 미평가(NULL)로 둔다.

상신(REVIEW) 이후 모듈에 종속된 질문 버전·응답은 직접 덮어쓰지 않는다.

평가 작성 시 버전별 질문·규칙 정의를 복사하고 version 값이 question_set_version 및 rule_version과 일치하는지 서버에서 검증한다. 동일 버전 이름에 다른 정의를 등록하지 않는다. 새 전역 버전을 기존 평가에 자동 적용하지 않는다. 계산에는 저장한 정의를 사용하며 단순한 함수명이나 버전 문자열만으로 원문을 대체하지 않는다.

승인 모듈의 질문·규칙·응답을 변경하려면 모듈을 새 개정하고 프로세스 및 적용 정의를 복사한다. 과거 모듈에 속한 원문은 그대로 보존한다.

[↑ 맨 위로](#top)

---

## VA

<a id="table-vendor_audit"></a>
### 32. 공급업체 감사 평가 (`vendor_audit`)

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
| 468 | 감사 ID | `audit_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 공급업체 감사 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 469 | 개정 간 논리 ID | `audit_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 470 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 471 | 문서 번호 | `document_number` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `VA-001` |
| 472 | 공급업체명 | `vendor_name` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | 감사 대상 공급업체명 | `Sample Vendor` |
| 473 | 감사 일자 | `audit_date` | `date` | N | N | - | Y | - | N | N | N | Y | 실제 감사 수행일. 작성자가 입력하며 개정 시 이전 개정 값을 복사한다 | `2026-09-16` |
| 474 | 감사자 | `auditor_name` | `varchar(100)` | N | N | - | Y | - | N | N | Y | Y | 감사를 수행한 감사자의 입력 당시 표시명. 사용자 이름이 바뀌어도 이력을 보존하며 개정 시 이전 개정 값을 복사한다 | `홍길동 · reviewer@example.test` |
| 475 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 476 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 477 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 478 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 479 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 480 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 481 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 482 | 감사자 계정 ID | `auditor_user_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 감사를 수행한 감사자 계정. 작성자가 선택하며 개정 시 이전 개정 값을 복사한다 | `00000000-0000-0000-0000-000000000001` |
| 483 | 평가 첨부파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 선택 업로드 첨부파일. 파일명·크기·경로는 파일 자산에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 484 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 485 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 486 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 487 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 488 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 489 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 490 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_vendor_audit_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 document_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_vendor_audit_2` | UNIQUE | `audit_key, revision_number` | UNIQUE (audit_key, revision_number) |
| `ck_vendor_audit_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_vendor_audit_3` | UNIQUE | `audit_key, is_current_version` | UNIQUE (audit_key) WHERE is_current_version = TRUE |
| `ck_vendor_audit_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_vendor_audit_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_vendor_audit_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_vendor_audit_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

시스템명은 프로젝트가 연결한 시스템에서 조회한다. 첨부파일은 선택 항목이며 NULL을 허용한다.

같은 audit_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 document_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## URS

<a id="table-requirement"></a>
### 33. 사용자 요구사항 명세 (`requirement`)

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
| 491 | 요구사항 ID | `requirement_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 요구사항 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 492 | 개정 간 논리 ID | `requirement_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 493 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 494 | 항목 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `URS-001` |
| 495 | 카테고리 | `category` | `varchar(100)` | N | N | - | Y | - | N | Y | N | Y | URS 대분류(library_item.category와 같은 기준). 시스템 관리, 감사추적, 전자서명 등 | `전자서명` |
| 496 | 중분류 | `middle_category` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 중분류(library_item.middle_category와 같은 기준). 라이브러리·시스템 패키지에서 등록하면 원본 값을 복사한다 | - |
| 497 | 대상 | `target_type` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | URS 적용 대상(library_item.target_type과 같은 기준). 라이브러리·시스템 패키지에서 등록하면 원본 값을 복사한다 | - |
| 498 | 항목 / 기능 | `title` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 요구사항 항목명 및 주요 기능 | `전자서명 서명자·일시·의미 기록` |
| 499 | 요구사항 상세 | `requirement_text` | `text` | N | N | - | Y | - | N | N | N | Y | 요구사항 상세 명세 내용 | `전자서명 시 서명자 ID, 서명 일시...` |
| 500 | 수용 기준 | `acceptance_criteria` | `text` | N | N | - | N | - | N | N | N | Y | 화면 수용 기준 입력 | `정의한 권한별 접근이 제한된다` |
| 501 | 근거 규정 | `regulation` | `text` | N | N | - | N | - | N | N | N | Y | 수기 규정 근거 또는 마스터 미연결 원문. 정규 조항 연결은 requirement_regulation에서 관리 | `CSV 규정 근거 검토 메모` |
| 502 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 503 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 504 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 505 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 506 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 507 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 요구사항 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 508 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 509 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 510 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 511 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 512 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 513 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 514 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 515 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 516 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 517 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 518 | 기타 출처 식별값 | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지 또는 AI 초안의 식별값. 외부/다형 출처이므로 DB FK 아님 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_requirement_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_requirement_2` | UNIQUE | `requirement_key, revision_number` | UNIQUE (requirement_key, revision_number) |
| `ck_requirement_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_requirement_3` | UNIQUE | `requirement_key, is_current_version` | UNIQUE (requirement_key) WHERE is_current_version = TRUE |
| `ck_requirement_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_requirement_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_requirement_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_requirement_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `rule_requirement_6` | 업무 검증 | `source_type, source_library_id` | LIBRARY이면 source_library_id 필수 |
| `rule_requirement_7` | 업무 검증 | `category, middle_category, target_type` | 작성 중(DRAFT)에는 middle_category·target_type의 NULL을 허용하고, 상신(REVIEW) 시 대분류(category)·중분류(middle_category)·대상(target_type)이 모두 입력되어 있어야 한다. |
| `ck_requirement_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

라이브러리/시스템 패키지/AI에서 가져온 내용은 이 항목의 초안으로 복사하여 편집한다. 출처 변경이 승인된 업무 항목에 자동 반영되지 않는다.

URS 분류는 대분류(category)·중분류(middle_category)·대상(target_type)이며 library_item과 같은 기준을 쓴다. 라이브러리 또는 시스템 패키지에서 등록하면 세 값을 함께 복사하고, 시스템 패키지 그룹핑도 이 세 값을 기준으로 한다.

requirement_id는 특정 개정행 PK이다. DQ/FRA/시험/규정은 해당 개정행에 연결한다. requirement_key별 최신 초안과 최신 승인 개정을 구분한다.

규정 조항은 requirement_regulation으로 연결하고 개정 복제 시 인용도 복사한다. RTM의 연결 상태는 업무 관계와 판정 결과에서 조회한다.

같은 requirement_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## FDS

<a id="table-fds_spec"></a>
### 34. 기능 설계 명세서 (`fds_spec`)

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
| 519 | FDS ID | `fds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FDS 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 520 | 개정 간 논리 ID | `fds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 521 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 522 | FDS 번호 | `fds_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `FDS-001` |
| 523 | FDS 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 문서의 표시 제목으로 업로드 파일명을 사용 | `FDS_ver1.pdf` |
| 524 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 525 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 526 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 527 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 528 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 529 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 530 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 531 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 532 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FDS 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 533 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 534 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 535 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 536 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 537 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 538 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 539 | 업로드 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 해당 설계 개정에 업로드한 파일. 교체하면 새 개정행과 새 파일 ID 생성. 파일 검사 전에 만든 초안(DRAFT)은 NULL일 수 있으며 검사 통과 시 file_link_request(PRIMARY_FILE)에 따라 채운다. DRAFT가 아니면 필수(ck_fds_spec_file_required) | `00000000-0000-0000-0000-000000000001` |
| 540 | 업로드 시각 | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이 파일 개정을 업로드한 시각 | `2026-09-01T00:00:00Z` |
| 541 | 업로더 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_fds_spec_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 fds_no와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_fds_spec_2` | UNIQUE | `fds_key, revision_number` | UNIQUE (fds_key, revision_number) |
| `ck_fds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fds_spec_3` | UNIQUE | `fds_key, is_current_version` | UNIQUE (fds_key) WHERE is_current_version = TRUE |
| `ck_fds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_fds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fds_spec_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_fds_spec_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `ck_fds_spec_file_required` | CHECK | `status, file_id` | CHECK (status = 'DRAFT' OR file_id IS NOT NULL) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

F&DS는 하나의 수행 활동이며 문서 종류의 표시 순서는 FDS 다음 DDS이다. DDS에 FDS 선행 승인 또는 단일 부모 FDS를 필수로 강제하지 않는다.

파일 교체 이력은 같은 논리키의 이전 개정행에서 조회한다. DQ는 이 테이블 PK를 정확한 승인 파일 개정으로 참조한다.

업로드 직후 파일 검사가 끝나기 전에도 초안을 만들 수 있으며, 검사를 통과하면 Worker가 file_link_request에 따라 file_id를 채운다. 초안을 삭제(deleted_at)하면 그 초안을 대상으로 하는 미연결 예약을 같은 트랜잭션에서 CANCELLED로 바꾼다(rule_file_link_request_3).

같은 fds_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 fds_no는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## DDS

<a id="table-dds_spec"></a>
### 35. 상세 설계 명세서 (`dds_spec`)

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
| 542 | DDS ID | `dds_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DDS 문서 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 543 | 개정 간 논리 ID | `dds_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 544 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | DDS가 속한 Validation 프로젝트 | `00000000-0000-0000-0000-000000000001` |
| 545 | DDS 번호 | `dds_no` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 최초 승인 시 부여. 초안은 NULL, 같은 논리키의 개정은 번호 유지 | `DDS-001` |
| 546 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 문서의 표시 제목으로 업로드 파일명을 사용 | `DDS_ver1.pdf` |
| 547 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 548 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 549 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 550 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 551 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 552 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 553 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 작성 사용자 | `00000000-0000-0000-0000-000000000001` |
| 554 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | DDS 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 555 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DDS 최종 수정 사용자 | `00000000-0000-0000-0000-000000000001` |
| 556 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-02T10:00:00Z` |
| 557 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 558 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 559 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-02T10:00:00Z` |
| 560 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 561 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 562 | 업로드 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 해당 설계 개정에 업로드한 파일. 교체하면 새 개정행과 새 파일 ID 생성. 파일 검사 전에 만든 초안(DRAFT)은 NULL일 수 있으며 검사 통과 시 file_link_request(PRIMARY_FILE)에 따라 채운다. DRAFT가 아니면 필수(ck_dds_spec_file_required) | `00000000-0000-0000-0000-000000000001` |
| 563 | 업로드 시각 | `uploaded_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 이 파일 개정을 업로드한 시각 | `2026-09-01T00:00:00Z` |
| 564 | 업로더 ID | `uploaded_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_dds_spec_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 dds_no와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_dds_spec_2` | UNIQUE | `dds_key, revision_number` | UNIQUE (dds_key, revision_number) |
| `ck_dds_spec_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dds_spec_3` | UNIQUE | `dds_key, is_current_version` | UNIQUE (dds_key) WHERE is_current_version = TRUE |
| `ck_dds_spec_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_dds_spec_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dds_spec_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_dds_spec_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `ck_dds_spec_file_required` | CHECK | `status, file_id` | CHECK (status = 'DRAFT' OR file_id IS NOT NULL) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

F&DS는 하나의 수행 활동이며 문서 종류의 표시 순서는 FDS 다음 DDS이다. DDS에 FDS 선행 승인 또는 단일 부모 FDS를 필수로 강제하지 않는다.

파일 교체 이력은 같은 논리키의 이전 개정행에서 조회한다. DQ는 이 테이블 PK를 정확한 승인 파일 개정으로 참조한다.

업로드 직후 파일 검사가 끝나기 전에도 초안을 만들 수 있으며, 검사를 통과하면 Worker가 file_link_request에 따라 file_id를 채운다. 초안을 삭제(deleted_at)하면 그 초안을 대상으로 하는 미연결 예약을 같은 트랜잭션에서 CANCELLED로 바꾼다(rule_file_link_request_3).

같은 dds_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 dds_no는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## DQ

<a id="table-dq_assessment"></a>
### 36. 설계 적격성 평가 (`dq_assessment`)

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
| 565 | DQ ID | `dq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 566 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 567 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설계 적격성 평가 문서 제목 | `설계 적격성 평가 (URS → FDS/DDS 매핑)` |
| 568 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 569 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 570 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 571 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | DQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 572 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_dq_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### 업무 규칙

항목별 업무 승인/개정은 자식 테이블에서 관리하고 생성 문서의 번호·버전·승인 상태는 deliverable_document/deliverable_revision에서 관리한다. 헤더의 승인 상태를 하위 항목 전체에 일괄 복사하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-dq_item"></a>
### 37. DQ 상세 평가 항목 (`dq_item`)

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
| 573 | DQ 항목 ID | `dq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | DQ 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 574 | 개정 간 논리 ID | `dq_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 575 | DQ ID | `dq_id` | `uuid` | N | Y | `dq_assessment.dq_id` | Y | - | N | Y | N | Y | 상위 DQ 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 576 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | 매핑 대상 URS 식별자 (FK) | `00000000-0000-0000-0000-000000000001` |
| 577 | URS 번호 | `urs_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | requirement_id가 가리키는 승인 개정의 번호 스냅샷. 직접 입력하지 않음 | `URS-001` |
| 578 | URS 요구사항 | `urs_description` | `text` | N | N | - | Y | - | N | N | N | Y | requirement_id가 가리키는 승인 개정의 요구사항 내용 스냅샷. 직접 입력하지 않음 | `사용자 로그인 및 전자서명 기능` |
| 579 | 판정 | `result_status` | `varchar(10)` | N | N | - | N | - | N | N | N | Y | PASS / FAIL / NA. NA=해당 없음, 미판정은 NULL | `PASS` |
| 580 | 비고 | `remarks` | `text` | N | N | - | N | - | N | N | N | Y | 검토 관련 특이사항 및 비고 | `FDS 및 DDS 설계 반영 완료 확인` |
| 581 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 582 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 583 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 584 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 585 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 586 | FDS 승인 개정 ID | `fds_revision_id` | `uuid` | N | Y | `fds_spec.fds_id` | N | - | N | Y | N | Y | fds_spec.fds_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 587 | FDS 검토 코멘트 | `fds_comment` | `text` | N | N | - | N | - | N | N | N | Y | FDS 검토 코멘트 | - |
| 588 | DDS 승인 개정 ID | `dds_revision_id` | `uuid` | N | Y | `dds_spec.dds_id` | N | - | N | Y | N | Y | dds_spec.dds_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 589 | DDS 검토 코멘트 | `dds_comment` | `text` | N | N | - | N | - | N | N | N | Y | DDS 검토 코멘트 | - |
| 590 | Fail 사유 | `fail_reason` | `text` | N | N | - | N | - | N | N | N | Y | Fail 사유 | - |
| 591 | 판정 수행자 ID | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 592 | 판정 수행 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 판정 수행 시각 | `2026-09-01T00:00:00Z` |
| 593 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 594 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 595 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 596 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 597 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 598 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 599 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `DQ-001` |
| 600 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 601 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 602 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 603 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_dq_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_dq_item_2` | UNIQUE | `dq_item_key, revision_number` | UNIQUE (dq_item_key, revision_number) |
| `ck_dq_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_dq_item_3` | UNIQUE | `dq_item_key, is_current_version` | UNIQUE (dq_item_key) WHERE is_current_version = TRUE |
| `ck_dq_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_dq_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_dq_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_dq_item_6` | CHECK | `result_status` | CHECK (result_status IS NULL OR result_status IN ('PASS','FAIL','NA')) |
| `rule_dq_item_6` | 업무 검증 | `result_status, fail_reason` | FAIL이면 공백이 아닌 fail_reason 필수 |
| `rule_dq_item_7` | 업무 검증 | `result_status, executed_by, executed_at` | 승인 요청 시 result_status·executed_by·executed_at 필수이며 FDS/DDS 중 하나 이상 같은 프로젝트의 유효 승인 개정이어야 한다. requirement_id도 같은 프로젝트의 승인 개정이어야 한다. |
| `ck_dq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

fds_revision_id와 dds_revision_id는 승인된 설계 문서의 정확한 개정행을 참조한다. 파일명·문서번호·버전은 연결한 fds_spec/dds_spec에서 조회한다.

수행자는 executed_by로 저장하며 검토·승인자는 workflow_instance와 approval_action에서 조회한다. NA(해당 없음) 판정도 미판정과 구분하고 설계 연결을 확인한다. DQ FAIL은 시험 일탈을 자동 생성하지 않는다.

같은 dq_item_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## FRA

<a id="table-fra_assessment"></a>
### 38. 기능 위험평가 (`fra_assessment`)

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
| 604 | FRA ID | `fra_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 605 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | Y | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 606 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | FMEA 기반 기능 위험평가 문서 제목 | `FMEA 기반 기능 위험평가` |
| 607 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 608 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 609 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 610 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | FRA 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 611 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_fra_assessment_1` | UNIQUE | `project_id` | UNIQUE (project_id) |

#### 업무 규칙

항목별 업무 승인/개정은 자식 테이블에서 관리하고 생성 문서의 번호·버전·승인 상태는 deliverable_document/deliverable_revision에서 관리한다. 헤더의 승인 상태를 하위 항목 전체에 일괄 복사하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-fra_item"></a>
### 39. FRA 위험 상세 항목 (`fra_item`)

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
| 612 | 위험 항목 ID | `fra_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | FRA 위험 세부 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 613 | 개정 간 논리 ID | `fra_item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 개정 간 유지하는 식별자. 개정행 PK와 구분 | `00000000-0000-0000-0000-000000000001` |
| 614 | FRA ID | `fra_id` | `uuid` | N | Y | `fra_assessment.fra_id` | Y | - | N | Y | N | Y | 상위 FRA 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 615 | URS 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | N | - | N | Y | N | Y | 동일 프로젝트의 승인된 URS 개정. 초안 NULL 허용, 승인 요청 전 필수 | `00000000-0000-0000-0000-000000000001` |
| 616 | URS 참조번호 | `urs_no` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 연결한 요구사항 개정의 승인번호 표시 스냅샷. requirement_id와 일치하며 독립 입력하지 않음 | `URS-001` |
| 617 | 기능명 | `feature_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 위험 평가 대상 기능명 | `전자서명` |
| 618 | 위험 시나리오 | `risk_scenario` | `text` | N | N | - | Y | - | N | N | N | Y | FMEA 위험 발생 시나리오 설명 | `전자서명 시 비밀번호 검증 미수행` |
| 619 | 승인 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중) / REVIEW(검토중) / APPROVAL(승인중) / APPROVED(승인완료) / REJECTED(반려) / CANCELLED(취소). REJECTED·CANCELLED는 이 개정의 종결 상태이며 이후 수정은 새 개정으로 처리 | `DRAFT` |
| 620 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 621 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 622 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 623 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 624 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 승인 이력 있는 항목은 삭제 대신 폐기 승인 처리 | `2026-09-01T00:00:00Z` |
| 625 | 심각도 SEV | `severity` | `smallint` | N | N | - | N | - | N | N | N | Y | 화면 입력 1~5. 미입력은 NULL이며 FRA 항목 상신 시 필수 | `5` |
| 626 | 발생가능성 OCC | `occurrence` | `smallint` | N | N | - | N | - | N | N | N | Y | 화면 입력 1~5. 미입력은 NULL이며 FRA 항목 상신 시 필수 | `2` |
| 627 | 검출도 DET | `detectability` | `varchar(1)` | N | N | - | N | - | N | N | N | Y | H / M / L. 미입력은 NULL이며 FRA 항목 상신 시 필수 | `M` |
| 628 | 판정 규칙 버전 | `rule_version` | `varchar(50)` | N | N | - | Y | `'UI-FRA-1'` | N | N | N | Y | 판정 규칙 버전 | `UI-FRA-1` |
| 629 | 승인 시 계산 결과 | `risk_result_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | 승인 당시 RP·RC·RPG·NT·Action Plan 자동 결과. 직접 입력하지 않음 | `{"RP":10,"RC":1,"RPG":"H","NT":"N","actionPlan":"Test 수행"}` |
| 630 | SOP 근거 조항 ID | `sop_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | N | - | N | Y | N | Y | RTM 화면의 SOP 연결을 원본 위험 항목에 저장 | `00000000-0000-0000-0000-000000000001` |
| 631 | 수기 SOP 참조 | `sop_reference_note` | `text` | N | N | - | N | - | N | N | N | Y | 조항 마스터에 아직 없는 SOP 참조 원문 | - |
| 632 | 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 해당 개정의 화면 표시 버전 | `ver1` |
| 633 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 논리키별 1부터 증가 | `1` |
| 634 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 시 입력하는 변경 사유. revision_number ≥ 2이면 필수 | `요구사항 변경` |
| 635 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 논리키의 최신 개정. 최신 승인본 여부와 구분 | `TRUE` |
| 636 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정의 업무 승인 절차. 검토·승인자와 서명은 연결된 절차/처리 이력에서 조회 | `00000000-0000-0000-0000-000000000001` |
| 637 | 항목 승인 번호 | `item_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 부여. 초안은 NULL, 개정 간 같은 번호 유지 | `FRA-001` |
| 638 | 사용 상태 | `lifecycle_status` | `varchar(20)` | N | N | - | Y | `'ACTIVE'` | N | N | N | Y | ACTIVE / DISPOSED. 승인 상태와 별개 | `ACTIVE` |
| 639 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 승인 완료 시각 | `2026-09-01T00:00:00Z` |
| 640 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 폐기 사유 | - |
| 641 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 642 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL / LIBRARY / SYSTEM_PACKAGE / AI_DRAFT | `MANUAL` |
| 643 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 644 | 기타 출처 식별값 | `source_reference` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지 또는 AI 초안의 식별값. 외부/다형 출처이므로 DB FK 아님 | - |
| 645 | 적용 위험 판정 규칙 원문 | `rule_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | {version,input_ranges,formulas,thresholds,matrix,action_mapping}. SEV/OCC/DET 허용값, RP·RC 계산식, RPG·NT 판정표와 Action Plan 매핑 전체 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_fra_item_1` | 업무 검증 | `workflow_instance_id` | APPROVED에는 item_number와 workflow_instance_id 필수. 같은 프로젝트 안의 서로 다른 논리키에 동일 승인 번호를 배정하지 않는다. |
| `uq_fra_item_2` | UNIQUE | `fra_item_key, revision_number` | UNIQUE (fra_item_key, revision_number) |
| `ck_fra_item_2_2` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `uq_fra_item_3` | UNIQUE | `fra_item_key, is_current_version` | UNIQUE (fra_item_key) WHERE is_current_version = TRUE |
| `ck_fra_item_4` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_fra_item_5` | CHECK | `lifecycle_status` | CHECK (lifecycle_status IN ('ACTIVE','DISPOSED')) |
| `ck_fra_item_disposal` | CHECK | `lifecycle_status, disposed_at, disposal_reason, disposal_workflow_id` | CHECK (lifecycle_status <> 'DISPOSED' OR (disposed_at IS NOT NULL AND disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL)) |
| `ck_fra_item_6` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `rule_fra_item_6` | 업무 검증 | `source_type, source_library_id` | LIBRARY이면 source_library_id 필수 |
| `ck_fra_item_7` | CHECK | `severity` | CHECK (severity IS NULL OR severity BETWEEN 1 AND 5) |
| `ck_fra_item_7_2` | CHECK | `occurrence` | CHECK (occurrence IS NULL OR occurrence BETWEEN 1 AND 5) |
| `ck_fra_item_7_3` | CHECK | `detectability` | CHECK (detectability IS NULL OR detectability IN ('H','M','L')) |
| `rule_fra_item_8` | 업무 검증 | `requirement_id, risk_result_snapshot` | 승인 요청 전 requirement_id 필수이고 해당 개정이 같은 프로젝트의 유효 승인본인지 검증한다. APPROVED에는 workflow_instance_id와 risk_result_snapshot 필수. |
| `rule_fra_item_9` | 업무 검증 | - | sop_clause_id는 INTERNAL_SOP 유형의 조항이어야 한다. |
| `rule_fra_item_10` | 업무 검증 | `severity, occurrence, detectability` | 작성 중(DRAFT)에는 미입력(NULL)을 허용하고 상신(REVIEW) 시 severity·occurrence·detectability가 모두 입력되어 있어야 한다. 미입력 값이 있으면 RP·RC·RPG·NT·Action Plan을 계산하지 않고 미평가로 표시하며 상신을 거부한다. |
| `ck_fra_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

각 행은 특정 개정이다. 수정 전 승인행은 보존하고 새 PK를 발급하여 개정하며 논리키를 유지한다. is_current_version은 최신 개정 여부이고 최신 승인본 여부와 다르다. 참조 FK와 전자서명은 정확한 개정행 PK 및 version을 가리킨다.

라이브러리/시스템 패키지/AI에서 가져온 내용은 이 항목의 초안으로 복사하여 편집한다. 출처 변경이 승인된 업무 항목에 자동 반영되지 않는다.

위험 평가 계산 규칙: RP=SEV×OCC; RC는 RP<5→3, RP<10→2, 나머지→1. RPG는 RC1의 DET H/M/L→M/H/H, RC2→L/M/H, RC3→L/L/M. NT는 RP>24→Y, 그 외 N. Action Plan은 RPG L→No Action, M→SOP 수정/삭제, H→Test 수행.

초안 결과는 조회 계산하고 승인 당시 결과만 스냅샷으로 보존한다. 위험 점수와 자동 조치계획은 판정 규칙으로 산출하며 독립 입력값으로 저장하지 않는다. 시험은 실제 시험/FRA 연결 관계로 추적한다.

작성 시 rule_version의 실제 정의를 rule_snapshot에 복사하고 해당 정의로 결과를 계산한다. 승인 후 규칙·입력·결과를 변경하지 않으며 변경 시 새 항목 개정을 생성한다. 같은 버전 이름의 정의는 변경하지 않는다. FRA 문서 상신 시 근거 항목별 정의와 입력·risk_result_snapshot을 evaluation_definition_snapshot에 고정한다.

같은 fra_item_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 item_number는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

## IQ

<a id="table-iq_assessment"></a>
### 40. 설치 적격성 평가 (`iq_assessment`)

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
| 646 | IQ ID | `iq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 647 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 648 | IQ 번호 | `iq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | IQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `IQ-VP-SYS-010-20260529` |
| 649 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 설치 적격성 평가 문서 제목 | `Installation Qualification` |
| 650 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 651 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | IQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 652 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | IQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 653 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 IQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 654 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 655 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | IQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 656 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 657 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | IQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 658 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 659 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | IQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 660 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각. 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나 상신된 문서·전자서명이 참조하는 평가 개정은 삭제할 수 없다(rule_iq_assessment_3) | `2026-09-01T00:00:00Z` |
| 661 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"iq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_assessment_1` | UNIQUE | `project_id, iq_no, revision_number` | UNIQUE(project_id, iq_no, revision_number) |
| `rule_iq_assessment_1` | 업무 검증 | `project_id, iq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_iq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_iq_assessment_current` | UNIQUE | `project_id, iq_no` | UNIQUE (project_id, iq_no) WHERE is_current_version=TRUE |
| `rule_iq_assessment_3` | 업무 검증 | `deleted_at` | 평가 개정의 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나, 상신된 deliverable_revision의 source_refs 또는 전자서명이 이 평가 개정을 참조하면 deleted_at을 설정할 수 없다. |
| `ck_iq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_iq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_iq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 iq_item의 실제 개정이며 같은 프로젝트·IQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 iq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다. 집계 대상은 구성(item_revision_refs)이 가리키는 시험 개정 중 폐기(is_disposed=TRUE)되지 않은 것이다. protocol_status는 각 시험 개정의 protocol_status를, record_status는 각 시험 개정에서 최초 수행 행(record_revision=1)이 중단(ABORTED)되지 않은 마지막 attempt_no의 유효 결과(is_current_revision=TRUE) record_status를 사용하며 수행이 없으면 DRAFT로 본다. 하나라도 REJECTED이면 REJECTED를 표시하고, 그렇지 않으면 CANCELLED를 DRAFT로 간주하여 DRAFT < REVIEW < APPROVAL < APPROVED 중 가장 덜 진행된 상태를 표시한다. 집계 대상이 없으면 DRAFT이다.

[↑ 맨 위로](#top)

---

<a id="table-iq_item"></a>
### 41. IQ 상세 테스트 항목 (`iq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `iq_item_id` |
| 주요 참조(FK) | `iq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 662 | IQ 항목 ID | `iq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | IQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 663 | IQ ID | `iq_id` | `uuid` | N | Y | `iq_assessment.iq_id` | Y | - | N | Y | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 664 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 665 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `IQ-NEW-01` |
| 666 | 테스트 케이스 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. 별도 분류값이 아님. | `하드웨어 설치` |
| 667 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `설치될 서버의 하드웨어 사양이 URS를 충족하는지 확인` |
| 668 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `하드웨어 사양이 URS에 명시된 요구사항과 일치해야 함` |
| 669 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 이 시험 개정의 독립적인 승인 상태이며 상위 문서 상태로 덮어쓰지 않음. | `DRAFT` |
| 670 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 671 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 672 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 673 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 674 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 상신 이력이 없는 DRAFT 개정만 삭제할 수 있으며, 승인 이력이 있는 항목(같은 item_key에 APPROVED 개정 존재)은 삭제 대신 폐기(is_disposed, rule_iq_item_7)로 처리한다 | `2026-09-01T00:00:00Z` |
| 675 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 676 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 677 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 678 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 679 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 680 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), SYSTEM_PACKAGE(시스템 패키지), AI_DRAFT(AI 초안). requirement·fra_item.source_type과 같은 코드 | `MANUAL` |
| 681 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 682 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `IQ-CORE` |
| 683 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 684 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `IQ-PKG-001` |
| 685 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 686 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 687 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 688 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 폐기 결재(approval_scope=DISPOSAL). 폐기는 이 결재의 최종 승인으로 완료한다 | `00000000-0000-0000-0000-000000000001` |
| 689 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 결재(disposal_workflow_id)의 최종 승인 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 690 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 691 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 폐기 결재의 최종 승인자 | `00000000-0000-0000-0000-000000000001` |
| 692 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 결재의 최종 승인 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_iq_item_1` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 IQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_iq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 상신(REVIEW) 이후의 프로토콜과 절차는 직접 수정하지 않으며, 반려·취소 후 수정과 승인 후 변경은 새 개정 행으로 처리한다. |
| `rule_iq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_iq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. SYSTEM_PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_iq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → IQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_iq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_iq_item_7` | 업무 검증 | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | 폐기는 폐기 권한자가 사유를 입력해 폐기 결재(approval_scope=DISPOSAL)를 상신하고 최종 승인으로 완료한다. is_disposed = TRUE이면 disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at 필수이며 disposal_signature_id·disposed_by·disposed_at은 그 결재의 최종 승인 서명·승인자·시각과 일치해야 한다. |
| `ck_iq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_iq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_iq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_iq_item_9` | 업무 검증 | `deleted_at` | deleted_at은 상신 이력이 없는 DRAFT 개정에만 설정할 수 있다. 같은 item_key에 APPROVED 개정이 있으면 삭제할 수 없으며 사용 중지는 폐기(is_disposed, rule_iq_item_7)로 처리한다. |
| `ck_iq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_iq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_iq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 iq_step, 실제 결과·서명·재수행 이력은 iq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-iq_step"></a>
### 42. IQ 시험 절차 (`iq_step`)

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
| 693 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 694 | 시험 개정 ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | iq_item.iq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 695 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 696 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 697 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 698 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 699 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 700 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_step_1` | UNIQUE | `iq_item_id, step_order` | UNIQUE(iq_item_id, step_order) |
| `rule_iq_step_1` | 업무 검증 | `iq_item_id, step_order` | step_order >= 1 |
| `rule_iq_step_2` | 업무 검증 | - | 상위 프로토콜이 상신(REVIEW)된 이후에는 절차 구조·본문을 변경하지 않는다. 반려·취소 후 수정과 승인 후 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 iq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-iq_execution"></a>
### 43. IQ 시험 수행 (`iq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | IQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `iq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 701 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 702 | 수행 프로토콜 개정 ID | `iq_item_id` | `uuid` | N | Y | `iq_item.iq_item_id` | Y | - | N | Y | N | Y | iq_item.iq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 703 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 704 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 705 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. 작성 중이거나 취소된 결과 정정 행은 FALSE이며 정정을 RECORDED로 확정할 때 전환한다. | `TRUE` |
| 706 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 707 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA(해당 없음). 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 708 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 709 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 710 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 711 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 712 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록), ABORTED(결과 미등록 중단. 결과 정정 행에서는 작성 중 정정 취소). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 713 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 프로토콜 승인 상태와 독립적으로 관리. 반려·취소 후 정정은 같은 attempt_no의 새 record_revision으로 처리. | `DRAFT` |
| 714 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 715 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 716 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 717 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 718 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 719 | 중단 사유 | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | 결과를 등록하지 않고 수행을 중단하거나 작성 중인 결과 정정을 취소(ABORTED)한 사유. ABORTED에서 필수 | - |
| 720 | 중단 서명 ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 중단 또는 결과 정정 취소 시 담당자가 중단 사유를 확인하고 입력한 전자서명(signature_action=ABORT, 서명 대상은 이 execution_id). 결과 등록 서명(execution_signature_id)과 구분하며 ABORTED에서 필수 | `00000000-0000-0000-0000-000000000001` |
| 721 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 722 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 723 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 724 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 725 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_execution_1` | UNIQUE | `iq_item_id, attempt_no, record_revision` | UNIQUE(iq_item_id, attempt_no, record_revision) |
| `rule_iq_execution_1` | 업무 검증 | `iq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개(uq_iq_execution_current로 강제) |
| `rule_iq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_iq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수. 같은 논리 시험(item_key)에 종료되지 않은 deviation(deviation_status<>'CLOSED')이 없으면 새 deviation을 만들고 source_entity_id = 이 execution_id로 한다. 진행 중인 deviation이 있으면(승인된 조치에 따른 재수행의 FAIL) 새 deviation을 만들지 않고 그 일탈에 다음 round_number의 deviation_action_round를 만들며 failed_execution_id = 이 execution_id로 한다. 일탈 상태 전환은 rule_deviation_9를 따른다. |
| `rule_iq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 같은 논리 시험에 종료되지 않은 deviation이 있으면 새 attempt_no는 그 일탈의 재수행으로만 시작할 수 있으며, deviation_status=RERUN_ALLOWED이고 현재 조치 회차(current_action_round_id)의 rerun_execution_id가 비어 있어야 한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_iq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록(RECORDED) 후에 가능하며 연결된 일탈의 상태와 무관하게 진행한다. FAIL 결과도 일탈 종결 전에 승인할 수 있다. 결과 승인은 판정이 올바르게 기록되었음을 확인하는 것이며 일탈 처리 완료를 뜻하지 않는다. 일탈 쪽의 선행 조건은 rule_deviation_4를 따른다. |
| `ck_iq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_iq_execution_6` | 업무 검증 | `qualification_result, record_revision` | 같은 attempt_no의 결과 정정(record_revision 증가): 정정 전 결과가 이미 일탈 또는 조치 회차에 연결된 FAIL이면 정정 결과가 FAIL이어도 새 일탈·새 회차를 만들지 않고 기존 연결을 유지하며 판정은 최신 유효 정정본으로 조회한다. 정정 전 PASS·NA였던 결과가 정정으로 FAIL이 되면 rule_iq_execution_3에 따라 처리한다. FAIL을 PASS로 정정해도 연결된 일탈과 이미 생성된 조치 회차는 그대로 두고, 그 회차 승인 후 재수행 없이 종료(rule_deviation_5)하는 등 일탈 절차로 종결한다. 완료보고 결재가 진행 중이면 재수행 결과를 정정하지 않는다. |
| `ck_iq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_iq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_iq_execution_7` | 업무 검증 | `qualification_result` | 종료되지 않은 일탈의 재수행 결과는 PASS 또는 FAIL만 등록한다(NA 불가). 재수행 중 시험이 해당 없다고 판단되면 재수행을 중단(rule_iq_execution_8)한 뒤 재수행 없이 종료(rule_deviation_5)하고 종료 사유에 근거를 기록한다. |
| `rule_iq_execution_8` | 업무 검증 | `execution_status, abort_reason` | 최초 수행 행(record_revision=1)이 IN_PROGRESS이면 결과를 등록하지 않고 중단(ABORTED)할 수 있다. 결과 정정 행(record_revision≥2)의 취소는 rule_iq_execution_9를 따른다. abort_reason과 중단 전자서명(abort_signature_id, signature_action=ABORT)이 필수이며 중단 처리자·시각은 그 서명의 signer_id·signed_at이다. 중단 후 이 행과 절차 수행·증적은 수정하지 않는다. ABORTED 수행은 결과 승인 대상이 아니며 다음 수행은 새 attempt_no로 시작한다. 중단한 수행이 일탈의 재수행이면 현재 조치 회차와 deviation의 rerun_execution_id 연결을 같은 트랜잭션에서 해제하고 deviation_status는 RERUN_ALLOWED를 유지한다. |
| `rule_iq_execution_9` | 업무 검증 | `record_revision, is_current_revision, execution_status` | 결과 정정 행(record_revision≥2)은 같은 attempt_no의 현재 유효 결과가 RECORDED일 때만 만들 수 있고, 같은 attempt_no에 작성 중인 정정 행은 최대 1개다. 정정 행은 IN_PROGRESS·is_current_revision=FALSE로 생성하며 그동안 이전 행이 현재 유효 결과로 남는다. 정정 행을 RECORDED로 확정(정정 서명)하는 트랜잭션에서 정정 행을 TRUE, 이전 행을 FALSE로 바꾸고 rule_iq_execution_6의 일탈 처리와 deviation.rerun_execution_id 갱신을 함께 수행한다. 작성 중인 정정을 취소하면 정정 행을 ABORTED(abort_reason과 중단 전자서명 abort_signature_id 필수)로 종결하고 is_current_revision=FALSE를 유지하며, 이전 행·재수행 연결·deviation 상태는 바꾸지 않는다. 다시 정정하려면 다음 record_revision으로 만든다. |
| `ck_iq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_iq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_iq_execution_current` | UNIQUE | `iq_item_id, attempt_no, is_current_revision` | UNIQUE (iq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_iq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 IQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다. 결과 등록·정정 확정 서명은 execution_signature_id(EXECUTE), 수행 중단·정정 취소 서명은 abort_signature_id(ABORT)에 연결한다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-iq_step_execution"></a>
### 44. IQ 절차 수행 (`iq_step_execution`)

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
| 726 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 727 | 수행 ID | `execution_id` | `uuid` | N | Y | `iq_execution.execution_id` | Y | - | N | Y | N | Y | iq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 728 | 절차 ID | `step_id` | `uuid` | N | Y | `iq_step.step_id` | Y | - | N | Y | N | Y | iq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 729 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 730 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 731 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 732 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 733 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 734 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 735 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_iq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_iq_step_execution_1` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_iq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 IQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## OQ

<a id="table-oq_assessment"></a>
### 45. 운전 적격성 평가 (`oq_assessment`)

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
| 736 | OQ ID | `oq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 737 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 738 | OQ 번호 | `oq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | OQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `OQ-VP-SYS-010-20260529` |
| 739 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 운전 적격성 평가 문서 제목 | `Operational Qualification` |
| 740 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 741 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | OQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 742 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | OQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 743 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 OQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 744 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 745 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | OQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 746 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 747 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | OQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 748 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 749 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | OQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 750 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각. 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나 상신된 문서·전자서명이 참조하는 평가 개정은 삭제할 수 없다(rule_oq_assessment_3) | `2026-09-01T00:00:00Z` |
| 751 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"oq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_assessment_1` | UNIQUE | `project_id, oq_no, revision_number` | UNIQUE(project_id, oq_no, revision_number) |
| `rule_oq_assessment_1` | 업무 검증 | `project_id, oq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_oq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_oq_assessment_current` | UNIQUE | `project_id, oq_no` | UNIQUE (project_id, oq_no) WHERE is_current_version=TRUE |
| `rule_oq_assessment_3` | 업무 검증 | `deleted_at` | 평가 개정의 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나, 상신된 deliverable_revision의 source_refs 또는 전자서명이 이 평가 개정을 참조하면 deleted_at을 설정할 수 없다. |
| `ck_oq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_oq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_oq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 oq_item의 실제 개정이며 같은 프로젝트·OQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 oq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다. 집계 대상은 구성(item_revision_refs)이 가리키는 시험 개정 중 폐기(is_disposed=TRUE)되지 않은 것이다. protocol_status는 각 시험 개정의 protocol_status를, record_status는 각 시험 개정에서 최초 수행 행(record_revision=1)이 중단(ABORTED)되지 않은 마지막 attempt_no의 유효 결과(is_current_revision=TRUE) record_status를 사용하며 수행이 없으면 DRAFT로 본다. 하나라도 REJECTED이면 REJECTED를 표시하고, 그렇지 않으면 CANCELLED를 DRAFT로 간주하여 DRAFT < REVIEW < APPROVAL < APPROVED 중 가장 덜 진행된 상태를 표시한다. 집계 대상이 없으면 DRAFT이다.

[↑ 맨 위로](#top)

---

<a id="table-oq_item"></a>
### 46. OQ 상세 테스트 항목 (`oq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `oq_item_id` |
| 주요 참조(FK) | `oq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 752 | OQ 항목 ID | `oq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | OQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 753 | OQ ID | `oq_id` | `uuid` | N | Y | `oq_assessment.oq_id` | Y | - | N | Y | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 754 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 755 | 테스트 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `OQ-AT-L01` |
| 756 | 테스트 케이스 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. 별도 분류값이 아님. | `Audit Trail` |
| 757 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 검증 수행 상세 절차 | `사용자 데이터 변경 시 감사추적 자동 생성` |
| 758 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `변경 전/후 값, 사용자, 날짜/시간, IP 기록` |
| 759 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 이 시험 개정의 독립적인 승인 상태이며 상위 문서 상태로 덮어쓰지 않음. | `DRAFT` |
| 760 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 761 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 762 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 763 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 764 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 상신 이력이 없는 DRAFT 개정만 삭제할 수 있으며, 승인 이력이 있는 항목(같은 item_key에 APPROVED 개정 존재)은 삭제 대신 폐기(is_disposed, rule_oq_item_7)로 처리한다 | `2026-09-01T00:00:00Z` |
| 765 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 766 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 767 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 768 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 769 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 770 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), SYSTEM_PACKAGE(시스템 패키지), AI_DRAFT(AI 초안). requirement·fra_item.source_type과 같은 코드 | `MANUAL` |
| 771 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 772 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `OQ-CORE` |
| 773 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 774 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `OQ-PKG-001` |
| 775 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 776 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 777 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 778 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 폐기 결재(approval_scope=DISPOSAL). 폐기는 이 결재의 최종 승인으로 완료한다 | `00000000-0000-0000-0000-000000000001` |
| 779 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 결재(disposal_workflow_id)의 최종 승인 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 780 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 781 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 폐기 결재의 최종 승인자 | `00000000-0000-0000-0000-000000000001` |
| 782 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 결재의 최종 승인 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_oq_item_1` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 OQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_oq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 상신(REVIEW) 이후의 프로토콜과 절차는 직접 수정하지 않으며, 반려·취소 후 수정과 승인 후 변경은 새 개정 행으로 처리한다. |
| `rule_oq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_oq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. SYSTEM_PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_oq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → OQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_oq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_oq_item_7` | 업무 검증 | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | 폐기는 폐기 권한자가 사유를 입력해 폐기 결재(approval_scope=DISPOSAL)를 상신하고 최종 승인으로 완료한다. is_disposed = TRUE이면 disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at 필수이며 disposal_signature_id·disposed_by·disposed_at은 그 결재의 최종 승인 서명·승인자·시각과 일치해야 한다. |
| `ck_oq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_oq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_oq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_oq_item_9` | 업무 검증 | `deleted_at` | deleted_at은 상신 이력이 없는 DRAFT 개정에만 설정할 수 있다. 같은 item_key에 APPROVED 개정이 있으면 삭제할 수 없으며 사용 중지는 폐기(is_disposed, rule_oq_item_7)로 처리한다. |
| `ck_oq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_oq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_oq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 oq_step, 실제 결과·서명·재수행 이력은 oq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-oq_step"></a>
### 47. OQ 시험 절차 (`oq_step`)

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
| 783 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 784 | 시험 개정 ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | oq_item.oq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 785 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 786 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 787 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 788 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 789 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 790 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_step_1` | UNIQUE | `oq_item_id, step_order` | UNIQUE(oq_item_id, step_order) |
| `rule_oq_step_1` | 업무 검증 | `oq_item_id, step_order` | step_order >= 1 |
| `rule_oq_step_2` | 업무 검증 | - | 상위 프로토콜이 상신(REVIEW)된 이후에는 절차 구조·본문을 변경하지 않는다. 반려·취소 후 수정과 승인 후 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 oq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-oq_execution"></a>
### 48. OQ 시험 수행 (`oq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | OQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `oq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 791 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 792 | 수행 프로토콜 개정 ID | `oq_item_id` | `uuid` | N | Y | `oq_item.oq_item_id` | Y | - | N | Y | N | Y | oq_item.oq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 793 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 794 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 795 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. 작성 중이거나 취소된 결과 정정 행은 FALSE이며 정정을 RECORDED로 확정할 때 전환한다. | `TRUE` |
| 796 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 797 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA(해당 없음). 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 798 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 799 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 800 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 801 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 802 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록), ABORTED(결과 미등록 중단. 결과 정정 행에서는 작성 중 정정 취소). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 803 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 프로토콜 승인 상태와 독립적으로 관리. 반려·취소 후 정정은 같은 attempt_no의 새 record_revision으로 처리. | `DRAFT` |
| 804 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 805 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 806 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 807 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 808 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 809 | 중단 사유 | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | 결과를 등록하지 않고 수행을 중단하거나 작성 중인 결과 정정을 취소(ABORTED)한 사유. ABORTED에서 필수 | - |
| 810 | 중단 서명 ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 중단 또는 결과 정정 취소 시 담당자가 중단 사유를 확인하고 입력한 전자서명(signature_action=ABORT, 서명 대상은 이 execution_id). 결과 등록 서명(execution_signature_id)과 구분하며 ABORTED에서 필수 | `00000000-0000-0000-0000-000000000001` |
| 811 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 812 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 813 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 814 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 815 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_execution_1` | UNIQUE | `oq_item_id, attempt_no, record_revision` | UNIQUE(oq_item_id, attempt_no, record_revision) |
| `rule_oq_execution_1` | 업무 검증 | `oq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개(uq_oq_execution_current로 강제) |
| `rule_oq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_oq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수. 같은 논리 시험(item_key)에 종료되지 않은 deviation(deviation_status<>'CLOSED')이 없으면 새 deviation을 만들고 source_entity_id = 이 execution_id로 한다. 진행 중인 deviation이 있으면(승인된 조치에 따른 재수행의 FAIL) 새 deviation을 만들지 않고 그 일탈에 다음 round_number의 deviation_action_round를 만들며 failed_execution_id = 이 execution_id로 한다. 일탈 상태 전환은 rule_deviation_9를 따른다. |
| `rule_oq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 같은 논리 시험에 종료되지 않은 deviation이 있으면 새 attempt_no는 그 일탈의 재수행으로만 시작할 수 있으며, deviation_status=RERUN_ALLOWED이고 현재 조치 회차(current_action_round_id)의 rerun_execution_id가 비어 있어야 한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_oq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록(RECORDED) 후에 가능하며 연결된 일탈의 상태와 무관하게 진행한다. FAIL 결과도 일탈 종결 전에 승인할 수 있다. 결과 승인은 판정이 올바르게 기록되었음을 확인하는 것이며 일탈 처리 완료를 뜻하지 않는다. 일탈 쪽의 선행 조건은 rule_deviation_4를 따른다. |
| `ck_oq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_oq_execution_6` | 업무 검증 | `qualification_result, record_revision` | 같은 attempt_no의 결과 정정(record_revision 증가): 정정 전 결과가 이미 일탈 또는 조치 회차에 연결된 FAIL이면 정정 결과가 FAIL이어도 새 일탈·새 회차를 만들지 않고 기존 연결을 유지하며 판정은 최신 유효 정정본으로 조회한다. 정정 전 PASS·NA였던 결과가 정정으로 FAIL이 되면 rule_oq_execution_3에 따라 처리한다. FAIL을 PASS로 정정해도 연결된 일탈과 이미 생성된 조치 회차는 그대로 두고, 그 회차 승인 후 재수행 없이 종료(rule_deviation_5)하는 등 일탈 절차로 종결한다. 완료보고 결재가 진행 중이면 재수행 결과를 정정하지 않는다. |
| `ck_oq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_oq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_oq_execution_7` | 업무 검증 | `qualification_result` | 종료되지 않은 일탈의 재수행 결과는 PASS 또는 FAIL만 등록한다(NA 불가). 재수행 중 시험이 해당 없다고 판단되면 재수행을 중단(rule_oq_execution_8)한 뒤 재수행 없이 종료(rule_deviation_5)하고 종료 사유에 근거를 기록한다. |
| `rule_oq_execution_8` | 업무 검증 | `execution_status, abort_reason` | 최초 수행 행(record_revision=1)이 IN_PROGRESS이면 결과를 등록하지 않고 중단(ABORTED)할 수 있다. 결과 정정 행(record_revision≥2)의 취소는 rule_oq_execution_9를 따른다. abort_reason과 중단 전자서명(abort_signature_id, signature_action=ABORT)이 필수이며 중단 처리자·시각은 그 서명의 signer_id·signed_at이다. 중단 후 이 행과 절차 수행·증적은 수정하지 않는다. ABORTED 수행은 결과 승인 대상이 아니며 다음 수행은 새 attempt_no로 시작한다. 중단한 수행이 일탈의 재수행이면 현재 조치 회차와 deviation의 rerun_execution_id 연결을 같은 트랜잭션에서 해제하고 deviation_status는 RERUN_ALLOWED를 유지한다. |
| `rule_oq_execution_9` | 업무 검증 | `record_revision, is_current_revision, execution_status` | 결과 정정 행(record_revision≥2)은 같은 attempt_no의 현재 유효 결과가 RECORDED일 때만 만들 수 있고, 같은 attempt_no에 작성 중인 정정 행은 최대 1개다. 정정 행은 IN_PROGRESS·is_current_revision=FALSE로 생성하며 그동안 이전 행이 현재 유효 결과로 남는다. 정정 행을 RECORDED로 확정(정정 서명)하는 트랜잭션에서 정정 행을 TRUE, 이전 행을 FALSE로 바꾸고 rule_oq_execution_6의 일탈 처리와 deviation.rerun_execution_id 갱신을 함께 수행한다. 작성 중인 정정을 취소하면 정정 행을 ABORTED(abort_reason과 중단 전자서명 abort_signature_id 필수)로 종결하고 is_current_revision=FALSE를 유지하며, 이전 행·재수행 연결·deviation 상태는 바꾸지 않는다. 다시 정정하려면 다음 record_revision으로 만든다. |
| `ck_oq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_oq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_oq_execution_current` | UNIQUE | `oq_item_id, attempt_no, is_current_revision` | UNIQUE (oq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_oq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 OQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다. 결과 등록·정정 확정 서명은 execution_signature_id(EXECUTE), 수행 중단·정정 취소 서명은 abort_signature_id(ABORT)에 연결한다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-oq_step_execution"></a>
### 49. OQ 절차 수행 (`oq_step_execution`)

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
| 816 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 817 | 수행 ID | `execution_id` | `uuid` | N | Y | `oq_execution.execution_id` | Y | - | N | Y | N | Y | oq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 818 | 절차 ID | `step_id` | `uuid` | N | Y | `oq_step.step_id` | Y | - | N | Y | N | Y | oq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 819 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 820 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 821 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 822 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 823 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 824 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 825 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_oq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_oq_step_execution_1` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_oq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 OQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## PQ

<a id="table-pq_assessment"></a>
### 50. 성능 적격성 평가 (`pq_assessment`)

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
| 826 | PQ ID | `pq_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 827 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 828 | PQ 번호 | `pq_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | PQ 문서 번호. 같은 문서의 개정 행에서는 동일 번호를 유지한다. | `PQ-VP-SYS-008-20260422` |
| 829 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 성능 적격성 평가 문서 제목 | `Performance Qualification` |
| 830 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 831 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | PQ 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 832 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | PQ 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 833 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 PQ 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 834 | 프로토콜 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ 프로토콜 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 835 | 레코드 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | PQ 수행 결과 승인 요약. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려). 구성에 속한 하위 상태를 업무 규칙의 집계 규칙으로 산출한 표시값이며 하위 행에 일괄 덮어쓰지 않음. 헤더는 결재 대상이 아니므로 CANCELLED를 두지 않는다. | `DRAFT` |
| 836 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 837 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PQ 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 838 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 839 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PQ 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 840 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각. 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나 상신된 문서·전자서명이 참조하는 평가 개정은 삭제할 수 없다(rule_pq_assessment_3) | `2026-09-01T00:00:00Z` |
| 841 | 평가 개정별 시험 구성 | `item_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | [{table_name:"pq_item",record_id,version,sort_order}] 배열. 이 평가 개정의 전체 시험 구성과 순서를 식별하며 이전 평가 개정의 변경 없는 시험 PK를 재사용할 수 있다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_assessment_1` | UNIQUE | `project_id, pq_no, revision_number` | UNIQUE(project_id, pq_no, revision_number) |
| `rule_pq_assessment_1` | 업무 검증 | `project_id, pq_no, revision_number, is_current_version` | 문서별 is_current_version = TRUE 행은 최대 1개 |
| `rule_pq_assessment_2` | 업무 검증 | - | 문서 개정과 시험 항목의 개정은 구분한다. 문서 화면의 승인 요약은 하위 프로토콜·수행 결과에서 산출한다. |
| `uq_pq_assessment_current` | UNIQUE | `project_id, pq_no` | UNIQUE (project_id, pq_no) WHERE is_current_version=TRUE |
| `rule_pq_assessment_3` | 업무 검증 | `deleted_at` | 평가 개정의 구성(item_revision_refs)에 상신 이력이 있는 시험 개정이 있거나, 상신된 deliverable_revision의 source_refs 또는 전자서명이 이 평가 개정을 참조하면 deleted_at을 설정할 수 없다. |
| `ck_pq_assessment_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_pq_assessment_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED')) |
| `ck_pq_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 목록, 프로토콜 승인, 결과 승인 및 문서 생성 정보를 관리한다. 문서 본문은 deliverable_document / deliverable_revision / deliverable_section에서 관리한다.

산출물 제목·본문·출력 형식만 변경할 때는 deliverable_revision만 새로 생성하고 평가 행과 시험 항목은 유지한다. 평가 범위·상위 업무 정보 또는 시험 구성이 변경되면 새 평가 개정 행을 만들고 item_revision_refs에 그 시점의 전체 구성을 고정한다. 변경 없는 시험 항목은 동일한 PK·버전을 다시 참조하며 복제하거나 상위 FK를 옮기지 않는다. 시험 프로토콜 내용이 바뀐 항목만 동일 item_key를 유지하는 새 개정행을 생성하여 새 구성에 포함한다.

구성의 대상은 pq_item의 실제 개정이며 같은 프로젝트·PQ 단계에 속해야 한다. 항목의 원본 상위 평가와 이 평가의 pq_no가 같아야 한다. 한 구성에 같은 item_key의 서로 다른 개정을 중복 포함하지 않고 record_id·sort_order도 중복을 금지한다. 구성은 정렬 순서를 포함한 전체 목록이며 특정 문서가 처음 상신될 때 잠근다. 상신된 문서 또는 과거 서명이 참조하는 평가 구성은 수정·삭제하지 않는다. 구성에 없는 행을 원본 상위 FK만으로 자동 포함하지 않는다. 프로토콜·결과 승인 요약은 해당 구성에 속한 항목과 수행에서 산출한다. 집계 대상은 구성(item_revision_refs)이 가리키는 시험 개정 중 폐기(is_disposed=TRUE)되지 않은 것이다. protocol_status는 각 시험 개정의 protocol_status를, record_status는 각 시험 개정에서 최초 수행 행(record_revision=1)이 중단(ABORTED)되지 않은 마지막 attempt_no의 유효 결과(is_current_revision=TRUE) record_status를 사용하며 수행이 없으면 DRAFT로 본다. 하나라도 REJECTED이면 REJECTED를 표시하고, 그렇지 않으면 CANCELLED를 DRAFT로 간주하여 DRAFT < REVIEW < APPROVAL < APPROVED 중 가장 덜 진행된 상태를 표시한다. 집계 대상이 없으면 DRAFT이다.

[↑ 맨 위로](#top)

---

<a id="table-pq_item"></a>
### 51. PQ 상세 테스트 항목 (`pq_item`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 시험 항목의 프로토콜 개정, 내용, 기대 결과, 허용 기준 및 등록 출처 관리 |
| Primary Key | `pq_item_id` |
| 주요 참조(FK) | `pq_id, created_by, updated_by, source_library_id, protocol_workflow_id, disposal_workflow_id, disposal_signature_id, disposed_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 842 | PQ 항목 ID | `pq_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | PQ 시험 항목의 특정 프로토콜 개정 행 PK. item_key는 개정 간 동일하게 유지한다. | `00000000-0000-0000-0000-000000000001` |
| 843 | PQ ID | `pq_id` | `uuid` | N | Y | `pq_assessment.pq_id` | Y | - | N | Y | N | Y | 이 논리 시험을 최초 등록한 상위 평가 개정. 후속 항목 개정에서도 동일한 원본 상위를 유지한다. 평가 개정별 실제 구성은 item_revision_refs에서 조회한다. | `00000000-0000-0000-0000-000000000001` |
| 844 | 시험 표시 ID | `test_id` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | 화면 시험 표시 ID. 동일 항목의 개정에서 같은 값을 유지하며, 다른 item_key 간 프로젝트 내 중복 금지. | `PQ-NEW-01` |
| 845 | 시험 항목명 | `test_case` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 화면의 테스트 항목 제목. | `업무 시나리오 검증` |
| 846 | 시험 항목 논리 ID | `item_key` | `uuid` | N | N | - | Y | `gen_random_uuid()` | N | Y | N | Y | 동일 시험 항목의 여러 개정을 묶는 식별자. 신규 항목에 한 번 발급하고 개정 시 유지. | `00000000-0000-0000-0000-000000000001` |
| 847 | 테스트 내용 | `test_description` | `text` | N | N | - | Y | - | N | N | N | Y | PQ 테스트 검증 수행 상세 절차 | `연속 3배치 이상 생산 공정 정상 완료 검증` |
| 848 | 기대 결과 | `expected_result` | `text` | N | N | - | Y | - | N | N | N | Y | 테스트 성공 기준 및 기대 결과 | `모든 배치가 사양에 맞게 정상 생산 완료되어야 함` |
| 849 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 850 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 851 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 852 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 853 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 미승인 초안 삭제 시각. 상신 이력이 없는 DRAFT 개정만 삭제할 수 있으며, 승인 이력이 있는 항목(같은 item_key에 APPROVED 개정 존재)은 삭제 대신 폐기(is_disposed, rule_pq_item_7)로 처리한다 | `2026-09-01T00:00:00Z` |
| 854 | 프로토콜 버전 | `version` | `varchar(20)` | N | N | - | Y | `'ver1'` | N | N | N | Y | 화면에 표시하는 프로토콜 개정 버전. | `ver1` |
| 855 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | item_key 내 개정 순번. 1 이상. | `1` |
| 856 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 프로토콜을 개정할 때 입력하는 변경 사유. | - |
| 857 | 최신 개정 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 동일 item_key의 최신 작업 개정 여부. 승인본 여부와 별개. | `TRUE` |
| 858 | 프로토콜 승인 상태 | `protocol_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소) | `DRAFT` |
| 859 | 허용 기준 | `acceptance_criteria` | `text` | N | N | - | Y | - | N | N | N | Y | 화면의 허용 기준. expected_result(기대 결과)와 구분. | `승인된 사양과 일치` |
| 860 | 등록 출처 | `source_type` | `varchar(30)` | N | N | - | Y | `'MANUAL'` | N | N | N | Y | MANUAL(직접 등록), LIBRARY(라이브러리), SYSTEM_PACKAGE(시스템 패키지), AI_DRAFT(AI 초안). requirement·fra_item.source_type과 같은 코드 | `MANUAL` |
| 861 | 원본 라이브러리 ID | `source_library_id` | `uuid` | N | Y | `library_item.library_id` | N | - | N | Y | N | Y | 라이브러리에서 등록한 경우의 원본. 시험 내용은 등록 시 복사되어 이후 독립적으로 관리. | `00000000-0000-0000-0000-000000000001` |
| 862 | 원본 패키지 코드 | `source_package_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 시스템 패키지에서 등록한 경우의 패키지 코드. | `PQ-CORE` |
| 863 | 원본 패키지 버전 | `source_package_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 등록 시 선택한 패키지 버전. | `ver2` |
| 864 | 원본 템플릿 코드 | `source_template_code` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 패키지 내부에서 선택한 시험 템플릿 코드. | `PQ-PKG-001` |
| 865 | 항목 승인번호 | `approval_number` | `varchar(50)` | N | N | - | N | - | N | N | N | Y | 최초 승인 시 발급해 화면 항목 번호로 표시. 개정 시 같은 논리 항목 번호를 유지. | - |
| 866 | 프로토콜 승인 워크플로우 | `protocol_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 프로토콜 개정에 대한 작성·검토·승인 경로 및 이력. | `00000000-0000-0000-0000-000000000001` |
| 867 | 폐기 사유 | `disposal_reason` | `text` | N | N | - | N | - | N | N | N | Y | 화면 폐기 승인에서 입력하는 사유. 폐기 완료 시 필수. | - |
| 868 | 폐기 승인 절차 ID | `disposal_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 폐기 결재(approval_scope=DISPOSAL). 폐기는 이 결재의 최종 승인으로 완료한다 | `00000000-0000-0000-0000-000000000001` |
| 869 | 폐기 승인 서명 | `disposal_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 폐기 결재(disposal_workflow_id)의 최종 승인 전자서명. | `00000000-0000-0000-0000-000000000001` |
| 870 | 폐기 여부 | `is_disposed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면의 폐기 상태. 폐기 항목은 신규 수행·집계 대상에서 제외하며 기존 기록은 보존. | `FALSE` |
| 871 | 폐기 처리자 | `disposed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 폐기 결재의 최종 승인자 | `00000000-0000-0000-0000-000000000001` |
| 872 | 폐기 시각 | `disposed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 폐기 결재의 최종 승인 시각. | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_item_1` | UNIQUE | `item_key, revision_number` | UNIQUE(item_key, revision_number) |
| `rule_pq_item_1` | 업무 검증 | `item_key, revision_number, is_current_version` | 같은 item_key는 동일 프로젝트와 PQ 단계에 속한다. is_current_version = TRUE 행은 item_key별 최대 1개 |
| `rule_pq_item_2` | 업무 검증 | - | protocol_status는 DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 상신(REVIEW) 이후의 프로토콜과 절차는 직접 수정하지 않으며, 반려·취소 후 수정과 승인 후 변경은 새 개정 행으로 처리한다. |
| `rule_pq_item_3` | 업무 검증 | - | approval_number는 논리 항목의 최초 승인 번호이며, 개정이 바뀌어도 유지한다. 승인·폐기 서명은 해당 항목 개정을 대상으로 한 workflow_instance / approval_action / electronic_signature에서 보존한다. |
| `rule_pq_item_4` | 업무 검증 | `source_library_id, source_package_code, source_package_version, source_template_code` | LIBRARY 등록이면 source_library_id 필수. SYSTEM_PACKAGE 등록이면 source_package_code / source_package_version / source_template_code 필수. |
| `rule_pq_item_5` | 업무 검증 | - | 복수 URS 연결은 traceability_link의 REQUIREMENT → PQ_ITEM / VERIFIED_BY로 관리한다. |
| `rule_pq_item_6` | 업무 검증 | - | 활성 시험 등록 및 프로토콜 승인에는 연결된 URS 개정이 최소 1개 필요하며 내용이 비어 있지 않은 세부 절차가 최소 1개 있어야 한다. |
| `rule_pq_item_7` | 업무 검증 | `disposal_reason, disposal_workflow_id, disposal_signature_id, is_disposed, disposed_by, disposed_at` | 폐기는 폐기 권한자가 사유를 입력해 폐기 결재(approval_scope=DISPOSAL)를 상신하고 최종 승인으로 완료한다. is_disposed = TRUE이면 disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at 필수이며 disposal_signature_id·disposed_by·disposed_at은 그 결재의 최종 승인 서명·승인자·시각과 일치해야 한다. |
| `ck_pq_item_disposal` | CHECK | `is_disposed, disposal_reason, disposal_workflow_id, disposal_signature_id, disposed_by, disposed_at` | CHECK (is_disposed = FALSE OR (disposal_reason IS NOT NULL AND length(trim(disposal_reason)) > 0 AND disposal_workflow_id IS NOT NULL AND disposal_signature_id IS NOT NULL AND disposed_by IS NOT NULL AND disposed_at IS NOT NULL)) |
| `rule_pq_item_8` | 업무 검증 | - | 폐기는 동일 item_key의 활성 여부에 적용하고 과거 승인 기록은 보존한다. 폐기 상태·사유·서명 메타데이터의 추가는 승인된 시험 본문·절차를 수정하는 개정과 구분하며, 원래 승인 내용은 변경하지 않는다. |
| `uq_pq_item_current` | UNIQUE | `item_key` | UNIQUE (item_key) WHERE is_current_version=TRUE |
| `rule_pq_item_9` | 업무 검증 | `deleted_at` | deleted_at은 상신 이력이 없는 DRAFT 개정에만 설정할 수 있다. 같은 item_key에 APPROVED 개정이 있으면 삭제할 수 없으며 사용 중지는 폐기(is_disposed, rule_pq_item_7)로 처리한다. |
| `ck_pq_item_source` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','LIBRARY','SYSTEM_PACKAGE','AI_DRAFT')) |
| `ck_pq_item_protocol_status` | CHECK | `protocol_status` | CHECK (protocol_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_pq_item_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 내용은 test_description, 기대 결과는 expected_result, 허용 기준은 acceptance_criteria에 저장한다. 반복 절차는 pq_step, 실제 결과·서명·재수행 이력은 pq_execution에서 관리한다.

같은 item_key의 모든 개정은 동일 프로젝트에 속한다. 항목 번호 approval_number·test_id는 최초 부여 후 후속 개정에서 유지한다. 승인 시 번호가 부여되는 항목의 미승인 초안은 NULL을 허용한다. 같은 프로젝트·같은 테이블의 다른 논리키에 해당 번호를 재사용하지 않으며 폐기 후에도 다른 항목에 재배정하지 않는다. 번호 발급과 개정 생성은 프로젝트 및 논리 항목에 대한 동시 처리 잠금으로 직렬화하고 같은 트랜잭션에서 중복·소속·번호 유지 조건을 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-pq_step"></a>
### 52. PQ 시험 절차 (`pq_step`)

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
| 873 | 절차 ID | `step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 874 | 시험 개정 ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | pq_item.pq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 875 | 절차 순서 | `step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 화면의 세부 절차 표시 순서. 1 이상. | `1` |
| 876 | 절차 내용 | `instruction` | `text` | N | N | - | Y | - | N | N | N | Y | 화면에서 추가·수정·삭제하는 절차 본문. | - |
| 877 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 878 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 879 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 880 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_step_1` | UNIQUE | `pq_item_id, step_order` | UNIQUE(pq_item_id, step_order) |
| `rule_pq_step_1` | 업무 검증 | `pq_item_id, step_order` | step_order >= 1 |
| `rule_pq_step_2` | 업무 검증 | - | 상위 프로토콜이 상신(REVIEW)된 이후에는 절차 구조·본문을 변경하지 않는다. 반려·취소 후 수정과 승인 후 변경은 새로운 시험 개정과 그 하위 절차로 기록한다. |

#### 업무 규칙

절차 완료 여부와 첨부 증적은 이 표의 설계값이 아니라 pq_step_execution의 수행 회차별 기록이다.

[↑ 맨 위로](#top)

---

<a id="table-pq_execution"></a>
### 53. PQ 시험 수행 (`pq_execution`)

| 항목 | 정의 |
|---|---|
| 설명 | PQ 시험 개정의 수행 회차별 실제 결과·판정·수행 서명·결과 승인 관리 |
| Primary Key | `execution_id` |
| 주요 참조(FK) | `pq_item_id, executed_by, execution_signature_id, abort_signature_id, result_workflow_id, created_by, updated_by, system_baseline_id` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 881 | 수행 ID | `execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 882 | 수행 프로토콜 개정 ID | `pq_item_id` | `uuid` | N | Y | `pq_item.pq_item_id` | Y | - | N | Y | N | Y | pq_item.pq_item_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 883 | 수행 회차 | `attempt_no` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 최초 수행 1, 재수행 2 이상. 화면 재수행 횟수는 attempt_no - 1. | `1` |
| 884 | 결과 정정 순번 | `record_revision` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 회차 결과를 정정한 이력의 순번. 재수행과 구분. | `1` |
| 885 | 최신 결과 정정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 시험 개정·회차에서 현재 유효한 결과 기록 여부. 작성 중이거나 취소된 결과 정정 행은 FALSE이며 정정을 RECORDED로 확정할 때 전환한다. | `TRUE` |
| 886 | 실제 결과 | `actual_result` | `text` | N | N | - | N | - | N | N | N | Y | 화면 실제 결과 입력. 판정 결과 등록 시 필수. | - |
| 887 | 수행 판정 | `qualification_result` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | PASS, FAIL, NA(해당 없음). 미실행은 NULL로 표시하며 승인 상태와 구분. | `PASS` |
| 888 | 수행일 | `performed_on` | `date` | N | N | - | N | - | N | N | N | Y | 화면에서 선택한 수행일. 전자서명 일시와 구분. | `2026-09-16` |
| 889 | 수행자 | `executed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 890 | 결과 등록 시각 | `executed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 전자서명을 통해 판정 결과를 등록한 시각. | `2026-09-01T00:00:00Z` |
| 891 | 수행 서명 ID | `execution_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 결과 등록 시 입력한 전자서명. 비밀번호는 저장하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 892 | 수행 진행 상태 | `execution_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | IN_PROGRESS(절차 수행·결과 입력 중), RECORDED(판정 결과 등록), ABORTED(결과 미등록 중단. 결과 정정 행에서는 작성 중 정정 취소). 일탈과 최종 승인은 별도 관리. | `IN_PROGRESS` |
| 893 | 결과 승인 상태 | `record_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 프로토콜 승인 상태와 독립적으로 관리. 반려·취소 후 정정은 같은 attempt_no의 새 record_revision으로 처리. | `DRAFT` |
| 894 | 결과 승인 워크플로우 | `result_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 해당 회차·결과 정정의 최종 결과 검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 895 | 결과 승인 버전 | `approval_version` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 최종 승인한 시험 결과의 표시 버전. 프로토콜 version과 구분. | `ver1` |
| 896 | 일탈 사유 | `deviation_reason` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 일탈 사유의 당시 값. | - |
| 897 | 즉시 조치 | `immediate_action` | `text` | N | N | - | N | - | N | N | N | Y | FAIL 결과를 등록할 때 입력한 즉시 조치의 당시 값. | - |
| 898 | 결과 정정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | record_revision이 증가할 때 결과를 정정한 이유. | - |
| 899 | 중단 사유 | `abort_reason` | `text` | N | N | - | N | - | N | N | N | Y | 결과를 등록하지 않고 수행을 중단하거나 작성 중인 결과 정정을 취소(ABORTED)한 사유. ABORTED에서 필수 | - |
| 900 | 중단 서명 ID | `abort_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 수행 중단 또는 결과 정정 취소 시 담당자가 중단 사유를 확인하고 입력한 전자서명(signature_action=ABORT, 서명 대상은 이 execution_id). 결과 등록 서명(execution_signature_id)과 구분하며 ABORTED에서 필수 | `00000000-0000-0000-0000-000000000001` |
| 901 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 902 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 903 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 904 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 905 | 수행 당시 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 수행 시작 시 프로젝트의 현재 기준을 고정한다. 결과 등록·서명 전 필수이며 이후 프로젝트 기준 변경으로 덮어쓰지 않는다. | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_execution_1` | UNIQUE | `pq_item_id, attempt_no, record_revision` | UNIQUE(pq_item_id, attempt_no, record_revision) |
| `rule_pq_execution_1` | 업무 검증 | `pq_item_id, attempt_no, record_revision, is_current_revision` | attempt_no / record_revision >= 1. 같은 시험 개정·회차의 is_current_revision = TRUE 행은 최대 1개(uq_pq_execution_current로 강제) |
| `rule_pq_execution_2` | 업무 검증 | `actual_result, qualification_result, performed_on, executed_by, executed_at, execution_signature_id` | 승인된 프로토콜에서만 수행 가능. RECORDED 전환 시 모든 절차 완료, actual_result / qualification_result / performed_on / executed_by / executed_at / execution_signature_id 필수. |
| `rule_pq_execution_3` | 업무 검증 | `qualification_result, deviation_reason, immediate_action` | qualification_result = FAIL이면 deviation_reason, immediate_action 필수. 같은 논리 시험(item_key)에 종료되지 않은 deviation(deviation_status<>'CLOSED')이 없으면 새 deviation을 만들고 source_entity_id = 이 execution_id로 한다. 진행 중인 deviation이 있으면(승인된 조치에 따른 재수행의 FAIL) 새 deviation을 만들지 않고 그 일탈에 다음 round_number의 deviation_action_round를 만들며 failed_execution_id = 이 execution_id로 한다. 일탈 상태 전환은 rule_deviation_9를 따른다. |
| `rule_pq_execution_4` | 업무 검증 | - | 재수행은 기존 결과·절차 완료·증적을 보존하고 새 attempt_no로 시작한다. 같은 논리 시험에 종료되지 않은 deviation이 있으면 새 attempt_no는 그 일탈의 재수행으로만 시작할 수 있으며, deviation_status=RERUN_ALLOWED이고 현재 조치 회차(current_action_round_id)의 rerun_execution_id가 비어 있어야 한다. 동일 회차 결과 정정은 record_revision을 증가시키며 이전 승인 기록을 덮어쓰지 않는다. |
| `rule_pq_execution_5` | 업무 검증 | - | 결과 최종 승인은 프로토콜 승인과 수행 결과 등록(RECORDED) 후에 가능하며 연결된 일탈의 상태와 무관하게 진행한다. FAIL 결과도 일탈 종결 전에 승인할 수 있다. 결과 승인은 판정이 올바르게 기록되었음을 확인하는 것이며 일탈 처리 완료를 뜻하지 않는다. 일탈 쪽의 선행 조건은 rule_deviation_4를 따른다. |
| `ck_pq_execution_baseline` | CHECK | `execution_status, system_baseline_id` | CHECK (execution_status <> 'RECORDED' OR system_baseline_id IS NOT NULL) |
| `rule_pq_execution_6` | 업무 검증 | `qualification_result, record_revision` | 같은 attempt_no의 결과 정정(record_revision 증가): 정정 전 결과가 이미 일탈 또는 조치 회차에 연결된 FAIL이면 정정 결과가 FAIL이어도 새 일탈·새 회차를 만들지 않고 기존 연결을 유지하며 판정은 최신 유효 정정본으로 조회한다. 정정 전 PASS·NA였던 결과가 정정으로 FAIL이 되면 rule_pq_execution_3에 따라 처리한다. FAIL을 PASS로 정정해도 연결된 일탈과 이미 생성된 조치 회차는 그대로 두고, 그 회차 승인 후 재수행 없이 종료(rule_deviation_5)하는 등 일탈 절차로 종결한다. 완료보고 결재가 진행 중이면 재수행 결과를 정정하지 않는다. |
| `ck_pq_execution_result` | CHECK | `qualification_result` | CHECK (qualification_result IS NULL OR qualification_result IN ('PASS','FAIL','NA')) |
| `ck_pq_execution_record_status` | CHECK | `record_status` | CHECK (record_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_pq_execution_7` | 업무 검증 | `qualification_result` | 종료되지 않은 일탈의 재수행 결과는 PASS 또는 FAIL만 등록한다(NA 불가). 재수행 중 시험이 해당 없다고 판단되면 재수행을 중단(rule_pq_execution_8)한 뒤 재수행 없이 종료(rule_deviation_5)하고 종료 사유에 근거를 기록한다. |
| `rule_pq_execution_8` | 업무 검증 | `execution_status, abort_reason` | 최초 수행 행(record_revision=1)이 IN_PROGRESS이면 결과를 등록하지 않고 중단(ABORTED)할 수 있다. 결과 정정 행(record_revision≥2)의 취소는 rule_pq_execution_9를 따른다. abort_reason과 중단 전자서명(abort_signature_id, signature_action=ABORT)이 필수이며 중단 처리자·시각은 그 서명의 signer_id·signed_at이다. 중단 후 이 행과 절차 수행·증적은 수정하지 않는다. ABORTED 수행은 결과 승인 대상이 아니며 다음 수행은 새 attempt_no로 시작한다. 중단한 수행이 일탈의 재수행이면 현재 조치 회차와 deviation의 rerun_execution_id 연결을 같은 트랜잭션에서 해제하고 deviation_status는 RERUN_ALLOWED를 유지한다. |
| `rule_pq_execution_9` | 업무 검증 | `record_revision, is_current_revision, execution_status` | 결과 정정 행(record_revision≥2)은 같은 attempt_no의 현재 유효 결과가 RECORDED일 때만 만들 수 있고, 같은 attempt_no에 작성 중인 정정 행은 최대 1개다. 정정 행은 IN_PROGRESS·is_current_revision=FALSE로 생성하며 그동안 이전 행이 현재 유효 결과로 남는다. 정정 행을 RECORDED로 확정(정정 서명)하는 트랜잭션에서 정정 행을 TRUE, 이전 행을 FALSE로 바꾸고 rule_pq_execution_6의 일탈 처리와 deviation.rerun_execution_id 갱신을 함께 수행한다. 작성 중인 정정을 취소하면 정정 행을 ABORTED(abort_reason과 중단 전자서명 abort_signature_id 필수)로 종결하고 is_current_revision=FALSE를 유지하며, 이전 행·재수행 연결·deviation 상태는 바꾸지 않는다. 다시 정정하려면 다음 record_revision으로 만든다. |
| `ck_pq_execution_status` | CHECK | `execution_status` | CHECK (execution_status IN ('IN_PROGRESS','RECORDED','ABORTED')) |
| `ck_pq_execution_abort` | CHECK | `execution_status, abort_reason, abort_signature_id, execution_signature_id` | CHECK (execution_status <> 'ABORTED' OR (abort_reason IS NOT NULL AND abort_signature_id IS NOT NULL AND execution_signature_id IS NULL)) |
| `uq_pq_execution_current` | UNIQUE | `pq_item_id, attempt_no, is_current_revision` | UNIQUE (pq_item_id, attempt_no) WHERE is_current_revision=TRUE |
| `ck_pq_execution_revision_reason` | CHECK | `record_revision, revision_reason` | CHECK (record_revision = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

시험 전체 증적은 evidence_link의 PQ_EXECUTION 대상으로 연결한다. 전자서명은 이 execution_id를 대상으로 하며 결과 정정 후 새 서명을 받는다. 결과 등록·정정 확정 서명은 execution_signature_id(EXECUTE), 수행 중단·정정 취소 서명은 abort_signature_id(ABORT)에 연결한다.

system_baseline_id는 해당 시험과 같은 프로젝트에 속해야 한다. 수행 시작 시 채택한 기준으로 프로토콜 승인과 선행 조건이 유효한지 검증한다. 재수행은 새 수행 기록에 당시 기준을 연결하며 과거 수행 기준은 유지한다. 결과 정정은 같은 논리 시험·attempt_no의 새 record_revision으로 기록하고 최초 수행의 기준을 유지한다.

[↑ 맨 위로](#top)

---

<a id="table-pq_step_execution"></a>
### 54. PQ 절차 수행 (`pq_step_execution`)

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
| 906 | 절차 수행 ID | `step_execution_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 907 | 수행 ID | `execution_id` | `uuid` | N | Y | `pq_execution.execution_id` | Y | - | N | Y | N | Y | pq_execution.execution_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 908 | 절차 ID | `step_id` | `uuid` | N | Y | `pq_step.step_id` | Y | - | N | Y | N | Y | pq_step.step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 909 | 절차 완료 여부 | `is_confirmed` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 화면 세부 절차의 완료/취소 값. | `FALSE` |
| 910 | 완료 처리자 | `confirmed_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 911 | 완료 처리 시각 | `confirmed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 절차를 완료 처리한 시각. | `2026-09-01T00:00:00Z` |
| 912 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 913 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 914 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 915 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_pq_step_execution_1` | UNIQUE | `execution_id, step_id` | UNIQUE(execution_id, step_id) |
| `rule_pq_step_execution_1` | 업무 검증 | `execution_id, step_id` | 절차와 수행은 반드시 같은 프로토콜 개정에 속해야 한다 |
| `rule_pq_step_execution_2` | 업무 검증 | `is_confirmed, confirmed_by, confirmed_at` | is_confirmed = TRUE이면 confirmed_by, confirmed_at 필수. 결과 등록 후 완료/취소와 증적 변경을 잠그고 과거 기록을 보존한다. |

#### 업무 규칙

절차 첨부 파일은 evidence_link의 PQ_STEP_EXECUTION / step_execution_id로 연결한다. 재수행 시 새 절차 수행 행을 생성한다.

[↑ 맨 위로](#top)

---

## VSR

<a id="table-vsr_assessment"></a>
### 55. 밸리데이션 종합 보고서 (`vsr_assessment`)

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
| 916 | VSR ID | `vsr_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR 평가 문서 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 917 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 연관 Validation 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 918 | VSR 번호 | `vsr_no` | `varchar(50)` | N | N | - | Y | - | N | N | N | Y | VSR 문서 번호. 개정 간 동일 번호를 유지. | `VSR-VP-SYS-008-20260422` |
| 919 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 밸리데이션 종합 보고서 제목 | `Validation Summary Report` |
| 920 | 밸리데이션 결론 | `overall_conclusion` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 밸리데이션 종합 결론. 선택 항목이며 업무 승인 상태와 구분한다. | - |
| 921 | 결론 상세 설명 | `conclusion_remarks` | `text` | N | N | - | N | - | N | N | N | Y | 밸리데이션 결론의 상세 설명과 조건사항. NULL을 허용한다. | `OQ-GMP-02 일탈 해결 완료 후 최종 승인 가능` |
| 922 | 문서 표시 버전 | `version` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 표시 버전 (예: v1.0, v1.1) | `v1.0` |
| 923 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | VSR 평가 문서 개정 순번 (예: 1, 2, 3...) | `1` |
| 924 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | VSR 평가 문서 신규 생성 및 개정 사유 | `최초 작성` |
| 925 | 현재 최신 버전 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 현재 활성화된 최신 VSR 평가 문서 버전 여부 (TRUE/FALSE) | `TRUE` |
| 926 | 문서 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | VSR 업무 확인 상태. DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). REJECTED·CANCELLED 후 수정은 새 VSR 개정으로 처리. 별도 생성 산출물의 승인 상태와 구분. | `DRAFT` |
| 927 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 928 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | VSR 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 929 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 930 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | VSR 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 931 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 932 | 원본 기준값 | `source_fingerprint` | `varchar(64)` | N | N | - | Y | - | N | N | N | Y | 프로젝트 선택 활동과 원본 개정 참조·요약값으로 계산한 SHA-256. 변경 시 재확인 대상 판별. | - |
| 933 | 집계 시각 | `captured_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 현재 VSR 활동 요약을 생성한 시각. | `2026-09-01T00:00:00Z` |
| 934 | VSR 확인 워크플로우 | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | VSR 업무 확인 요청·검토·승인 경로. | `00000000-0000-0000-0000-000000000001` |
| 935 | VSR 확인 서명 | `approval_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | VSR 최종 확인 완료 시 전자서명. | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vsr_assessment_1` | UNIQUE | `project_id, vsr_no, revision_number` | UNIQUE(project_id, vsr_no, revision_number) |
| `rule_vsr_assessment_1` | 업무 검증 | `project_id, vsr_no, revision_number` | 같은 문서의 최신 개정은 최대 1개(uq_vsr_assessment_current로 강제) |
| `rule_vsr_assessment_2` | 업무 검증 | - | 승인 완료된 VSR 헤더와 하위 요약값은 보존한다. 원본 변경 시 새 VSR 개정에 DRAFT로 재집계하여 확인을 받는다. |
| `rule_vsr_assessment_3` | 업무 검증 | - | VSR에는 프로젝트에서 선택한 수행 단계만 집계하며 RTM은 제외한다. VSR 자체 확인은 헤더에 표시하고 상세 요약에 재귀적으로 저장하지 않는다. |
| `ck_vsr_assessment_status` | CHECK | `status` | CHECK (status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `uq_vsr_assessment_current` | UNIQUE | `project_id, vsr_no, is_current_version` | UNIQUE (project_id, vsr_no) WHERE is_current_version=TRUE |
| `ck_vsr_assessment_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

화면 활동 요약의 원본 상태 확인용 기록이다. 보고서 본문·PDF 생성 및 해당 문서 승인은 deliverable_document / deliverable_revision에서 별도로 관리한다.

[↑ 맨 위로](#top)

---

<a id="table-vsr_item"></a>
### 56. VSR 활동 요약 항목 (`vsr_item`)

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
| 936 | VSR 항목 ID | `vsr_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | VSR 활동 요약 항목 식별자 (PK) | `00000000-0000-0000-0000-000000000001` |
| 937 | VSR ID | `vsr_id` | `uuid` | N | Y | `vsr_assessment.vsr_id` | Y | - | N | Y | N | Y | 상위 VSR 문서 식별자 | `00000000-0000-0000-0000-000000000001` |
| 938 | 활동 구분 | `activity_code` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 집계 활동 코드: VP, VA, QIA, URS, FDS_GROUP(화면 F&DS), FRA, DQ, IQ, OQ, PQ. RTM 제외; VSR 자기 상태는 헤더에서 표시. | `IQ` |
| 939 | 문서 번호 | `doc_no` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 집계 당시 표시한 해당 활동 문서 번호. 미승인·미생성 시 NULL. | `VP-SYS-008-20260422 · URS` |
| 940 | 개정 차수 | `revision_no` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 집계 당시 표시한 원본 승인 버전. 미승인·미생성 시 NULL. 특정 개정 식별은 source_revision_refs에 보존. | `ver1` |
| 941 | 일탈 건수 | `deviation_info` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 집계 당시 일탈 건수·미결 현황 표시 스냅샷. 독립적으로 수동 편집하지 않음. | - |
| 942 | 결론/상태 | `item_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | 집계 당시 활동 승인 상태: DRAFT(작성중), REVIEW(검토중), APPROVAL(승인중), APPROVED(승인완료), REJECTED(반려), CANCELLED(취소). 원본 업무 상태를 자동 조회한 스냅샷. | `APPROVED` |
| 943 | 승인자명 | `approver_name` | `varchar(50)` | N | N | - | N | - | N | N | Y | Y | 원본 승인 당시 승인자 표시명 스냅샷. | `홍길동` |
| 944 | 승인일 | `approval_date` | `date` | N | N | - | N | - | N | N | N | Y | 원본 승인 당시 화면 표시용 승인일 스냅샷. 정확한 서명 시각은 원본 기록에서 조회. | `2024-02-20` |
| 945 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 946 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 작성자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 947 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각 (UTC) | `2026-08-26T00:00:00Z` |
| 948 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 항목 수정자 식별자 | `00000000-0000-0000-0000-000000000001` |
| 949 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 950 | 프로젝트 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | 해당 VSR과 같은 프로젝트의 선택된 수행 활동. | `00000000-0000-0000-0000-000000000001` |
| 951 | 산출물 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | 산출물 문서가 생성된 경우 참조하는 개정. IQ/OQ/PQ는 document_type=RECORD, 그 외 활동은 STAGE_DELIVERABLE 문서의 개정이며 문서의 project_activity_id가 이 행과 같아야 한다. 업무 항목 승인만 있는 단계에서는 NULL 가능. | `00000000-0000-0000-0000-000000000001` |
| 952 | 원본 개정 참조 목록 | `source_revision_refs` | `jsonb` | N | N | - | Y | `'[]'::jsonb` | N | N | N | Y | 집계 근거인 정확한 업무 개정 목록: [{table_name, record_id, version}]. table_name은 실제 테이블명이며 IQ/OQ/PQ 결과는 iq_execution / oq_execution / pq_execution의 execution_id, FDS/DDS는 fds_spec.fds_id / dds_spec.dds_id 개정 PK 사용. 미승인 단계는 빈 배열. | `[]` |
| 953 | 원본 요약 스냅샷 | `source_snapshot` | `jsonb` | N | N | - | Y | `'{}'::jsonb` | N | N | Y | Y | 집계 당시 승인 표의 columns / rows와 승인정보. 원본 개정 참조와 함께 표시값을 고정하며 미승인 단계는 빈 객체. | `{}` |
| 954 | 활동 원본 기준값 | `source_fingerprint` | `varchar(64)` | N | N | - | N | - | N | N | N | Y | 해당 단계의 원본 참조와 스냅샷을 해시한 값. 승인 원본이 없으면 NULL. | - |
| 955 | 활동 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 프로젝트의 선택 활동 순서. | `1` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_vsr_item_1` | UNIQUE | `vsr_id, project_activity_id` | UNIQUE(vsr_id, project_activity_id) |
| `rule_vsr_item_1` | 업무 검증 | `vsr_id, project_activity_id` | activity_code와 project_activity의 활동 코드는 일치해야 한다 |
| `rule_vsr_item_2` | 업무 검증 | `project_activity_id, document_revision_id` | project_activity_id / document_revision_id / source_revision_refs는 같은 프로젝트를 가리켜야 한다. 참조 원본의 존재·유형·개정과 승인 여부를 검증한다. |
| `rule_vsr_item_3` | 업무 검증 | `item_status` | item_status = APPROVED이면 source_revision_refs가 비어 있지 않아야 하며 승인 시점의 source_snapshot과 표시값을 보존한다. |
| `rule_vsr_item_4` | 업무 검증 | - | 일탈 표시값은 해당 활동의 일탈 기록에서 집계한다. |
| `rule_vsr_item_5` | 업무 검증 | - | 상위 VSR 승인 후 상세행은 수정하지 않는다. 새 집계는 새 VSR 개정의 상세행으로 기록한다. |
| `ck_vsr_item_status` | CHECK | `item_status` | CHECK (item_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_vsr_item_6` | 업무 검증 | `document_revision_id, project_activity_id` | document_revision_id의 deliverable_document.project_activity_id는 이 행의 project_activity_id와 같아야 하며, IQ/OQ/PQ 활동은 document_type=RECORD, 그 외 활동은 STAGE_DELIVERABLE 문서의 개정만 참조한다. |

#### 업무 규칙

source_revision_refs는 실제 테이블명·개정 PK·버전의 다형 참조 목록으로 물리 FK가 아니다. deliverable_revision.source_refs와 동일한 객체 구조를 사용한다. 문서 파일 개정과 업무 확인 상태를 같은 것으로 취급하지 않는다.

[↑ 맨 위로](#top)

---

## Workflow

<a id="table-workflow_instance"></a>
### 57. Workflow 인스턴스 (`workflow_instance`)

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
| 956 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 문서별 검토·승인 Workflow 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 957 | 대상 테이블명 | `target_table_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 검토·승인 대상 테이블명. approval_scope별 허용 테이블(업무 규칙의 표)만 허용한다. | `fds_spec` |
| 958 | 대상 레코드 ID | `target_record_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | 대상 테이블 행 PK. 다형 참조이므로 실제 FK가 아니며 대상 존재·프로젝트·버전을 함께 검증한다. | `00000000-0000-0000-0000-000000000001` |
| 959 | 대상 문서 버전 | `target_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 승인 대상의 표시 버전 또는 개정번호. 인벤토리는 system_asset.revision_number, 종료 요청은 CLOSE-{request_version}에 대응 | `v1.0` |
| 960 | 상신자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 문서를 검토·승인 절차에 상신한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 961 | 상신 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 문서가 검토·승인 절차에 상신된 시각 | `2026-08-31T15:00:00Z` |
| 962 | Workflow 상태 | `workflow_status` | `varchar(20)` | N | N | - | Y | `'IN_PROGRESS'` | N | N | N | Y | 전체 Workflow 상태. IN_PROGRESS, APPROVED, REJECTED, CANCELLED. 인스턴스는 상신 시점에 IN_PROGRESS로 생성하며 작성중(DRAFT) 상태를 두지 않는다 | `IN_PROGRESS` |
| 963 | 현재 단계 순서 | `current_step_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 현재 처리 중인 검토·승인 단계 순서. 상신 시 최초 비삭제 단계의 step_order를 저장하며 1로 고정하지 않는다 | `1` |
| 964 | 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 승인·반려 또는 취소로 Workflow가 종료된 시각 | `2026-08-31T17:00:00Z` |
| 965 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00Z` |
| 966 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 레코드 생성자 | `00000000-0000-0000-0000-000000000001` |
| 967 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00Z` |
| 968 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 레코드 최종 수정자 | `00000000-0000-0000-0000-000000000001` |
| 969 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |
| 970 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 업무에 대한 결재이면 필수. 인벤토리 등 프로젝트 외 결재는 NULL | `00000000-0000-0000-0000-000000000001` |
| 971 | 승인 구분 | `approval_scope` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | ITEM/PROTOCOL/RESULT/DOCUMENT/INVENTORY/CLOSURE/DISPOSAL/DEVIATION_ACTION/DEVIATION_COMPLETION/DEVIATION_CLOSE | `DOCUMENT` |
| 972 | 적용 결재선 설정 ID | `workflow_config_id` | `uuid` | N | Y | `project_workflow_config.config_id` | N | - | N | Y | N | Y | 상신 때 사용한 프로젝트 결재선 설정. 인벤토리 자체 경로 등은 NULL | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_instance_1` | UNIQUE | `target_table_name, target_record_id, target_version, approval_scope, workflow_status` | UNIQUE (target_table_name, target_record_id, target_version, approval_scope) WHERE workflow_status = 'IN_PROGRESS' |
| `ck_workflow_instance_2` | CHECK | `workflow_status` | CHECK (workflow_status IN ('IN_PROGRESS','APPROVED','REJECTED','CANCELLED')) |
| `ck_workflow_instance_scope` | CHECK | `approval_scope` | CHECK (approval_scope IN ('ITEM','PROTOCOL','RESULT','DOCUMENT','INVENTORY','CLOSURE','DISPOSAL','DEVIATION_ACTION','DEVIATION_COMPLETION','DEVIATION_CLOSE')) |
| `rule_workflow_instance_3` | 업무 검증 | `approval_scope, target_table_name, target_version` | approval_scope와 target_table_name·target_version 형식은 업무 규칙의 범위별 대상 표와 일치해야 한다. |

#### 업무 규칙

상신할 때 설정의 단계·모드·주/대체 담당자를 workflow_step 및 workflow_step_assignee로 복사한다. 이후 기본 설정 변경이 이미 진행 중인 결재선을 바꾸지 않는다. 작성자·검토자·승인자와 대체자는 서로 다른 활성 계정으로 지정하고, 인벤토리도 작성·검토·승인자가 서로 다른지 검사한다.

approval_scope별 대상:

| approval_scope | target_table_name | target_version 형식 |
|---|---|---|
| ITEM | vp_plan, qia_module_item, vendor_audit, requirement, fds_spec, dds_spec, dq_item, fra_item, vsr_assessment | 대상 개정의 version |
| PROTOCOL | iq_item, oq_item, pq_item | 대상 시험 개정의 version |
| RESULT | iq_execution, oq_execution, pq_execution | {attempt_no}-{record_revision} |
| DOCUMENT | deliverable_revision | 대상 문서 개정의 version |
| INVENTORY | system_asset | system_asset.revision_number |
| CLOSURE | project_closure_request | CLOSE-{request_version} |
| DISPOSAL | vp_plan, qia_module_item, vendor_audit, requirement, fds_spec, dds_spec, dq_item, fra_item, iq_item, oq_item, pq_item | 폐기 대상 개정의 version |
| DEVIATION_ACTION | deviation_action_round | ACTION-{round_number}-{revision_number} |
| DEVIATION_COMPLETION | deviation_completion_revision (closure_kind=COMPLETION) | COMPLETE-{조치 회차}-{revision_number} |
| DEVIATION_CLOSE | deviation_completion_revision (closure_kind=CLOSE) | CLOSE-{조치 회차}-{revision_number} |

system_asset의 폐기와 결재선 적용(CONFIG_APPLY)은 Workflow 없이 해당 테이블의 전자서명으로 처리한다.

Workflow가 REJECTED 또는 CANCELLED로 종료되면 같은 트랜잭션에서 대상 개정의 승인 상태를 같은 값(REJECTED/CANCELLED)으로 전환한다. 폐기 결재(DISPOSAL)가 반려·취소되면 대상의 승인 상태는 바꾸지 않고 lifecycle_status를 ACTIVE로(IQ/OQ/PQ 시험 항목은 is_disposed=FALSE로) 유지한다. 종료 요청(CLOSURE)은 project_closure_request.status를 같은 값으로 전환한다. 일탈 조치(DEVIATION_ACTION)는 deviation_action_round.approval_status에, 일탈 완료보고(DEVIATION_COMPLETION)·미수행 종료(DEVIATION_CLOSE)는 deviation_completion_revision.approval_status에 위 일반 규칙을 적용한다. 두 종결 결재가 반려·취소되어도 deviation_status는 직전 상태(COMPLETION_PENDING 또는 RERUN_ALLOWED)를 유지하고 deviation의 서명·승인자 컬럼은 채우지 않는다.

하나의 시험 항목이라도 PROTOCOL 승인과 수행 RESULT 승인을 분리한다. 승인 범위·대상 테이블·PK·버전을 함께 식별한다. 작성자는 requested_by 및 SUBMIT 처리 이력으로 기록한다.

DEVIATION_ACTION은 deviation_action_round의 PK와 ACTION-{round_number}-{revision_number}를 대상으로 한다. DEVIATION_COMPLETION과 DEVIATION_CLOSE는 deviation_completion_revision의 PK와 COMPLETE-·CLOSE- 버전을 대상으로 한다. 반려·취소된 조치·종결 보고를 수정하여 재상신할 때 새 개정과 Workflow를 생성한다.

[↑ 맨 위로](#top)

---

<a id="table-workflow_step"></a>
### 58. Workflow 단계 (`workflow_step`)

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
| 973 | Workflow 단계 ID | `workflow_step_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | Workflow 검토·승인 단계 고유 식별자. deleted_at IS NULL인 행에 (workflow_instance_id, step_order) 중복을 허용하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 974 | Workflow 인스턴스 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | Y | - | N | Y | N | Y | 단계가 소속된 Workflow 식별자 | `00000000-0000-0000-0000-000000000001` |
| 975 | 단계 순서 | `step_order` | `integer` | N | N | - | Y | - | N | Y | N | Y | 동일 Workflow 내 검토·승인 처리 순서 | `1` |
| 976 | 단계 유형 | `step_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | 처리 단계 유형. REVIEW 또는 APPROVE. SUBMIT/CANCEL은 단계 유형이 아닌 처리 이력의 action_type으로 기록한다 | `REVIEW` |
| 977 | 단계명 | `step_name` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 화면에 표시할 검토·승인 단계명 | `품질 검토` |
| 978 | 처리 모드 | `execution_mode` | `varchar(20)` | N | N | - | Y | `'SERIAL'` | N | N | N | Y | SERIAL=직렬, PARALLEL=병렬. 동일 단계의 여러 담당자 처리 순서 | `SERIAL` |
| 979 | 단계 상태 | `step_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | 단계 상태. PENDING, IN_PROGRESS, APPROVED, REJECTED, SKIPPED | `PENDING` |
| 980 | 처리 기한 | `due_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계의 검토·승인 처리 예정 기한 | `2026-09-02T18:00:00Z` |
| 981 | 처리 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 해당 단계가 승인·반려 등으로 완료된 시각 | `2026-09-01T10:00:00Z` |
| 982 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-08-31T15:00:00Z` |
| 983 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 단계 생성자 | `00000000-0000-0000-0000-000000000001` |
| 984 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 최종 수정 시각(UTC) | `2026-08-31T16:00:00Z` |
| 985 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | Workflow 단계 최종 수정자 | `00000000-0000-0000-0000-000000000001` |
| 986 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 소프트 삭제 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_step_1` | UNIQUE | `workflow_instance_id, step_order, deleted_at` | UNIQUE (workflow_instance_id, step_order) WHERE deleted_at IS NULL |
| `ck_workflow_step_2` | CHECK | `execution_mode` | CHECK (execution_mode IN ('SERIAL','PARALLEL')) |
| `ck_workflow_step_3` | CHECK | `step_order` | CHECK (step_order >= 1) |
| `ck_workflow_step_type` | CHECK | `step_type` | CHECK (step_type IN ('REVIEW','APPROVE')) |
| `ck_workflow_step_status` | CHECK | `step_status` | CHECK (step_status IN ('PENDING','IN_PROGRESS','APPROVED','REJECTED','SKIPPED')) |

#### 업무 규칙

주 담당자·대체자 및 담당자 순서는 workflow_step_assignee에 저장한다. step_order는 단계 순서이며 execution_mode는 같은 단계 안의 담당자 처리 방식을 구분한다.

병렬 단계도 지정된 담당자들의 처리가 모두 완료되어야 다음 단계로 진행한다. 검토 단계는 REVIEW, 승인 단계는 APPROVE 처리 이력을 남긴다.

[↑ 맨 위로](#top)

---

<a id="table-workflow_step_assignee"></a>
### 59. 결재 단계 담당자 (`workflow_step_assignee`)

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
| 987 | 담당 배정 ID | `assignment_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 988 | 결재 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | workflow_step.workflow_step_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 989 | 주 담당자 ID | `assignee_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 990 | 대체 담당자 ID | `substitute_user_id` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 991 | 담당자 순서 | `assignee_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 직렬 단계에서 처리할 순서 | `1` |
| 992 | 처리 상태 | `assignment_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | N | N | Y | PENDING/IN_PROGRESS/APPROVED/REJECTED/SKIPPED | `PENDING` |
| 993 | 처리 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 처리 완료 시각 값 | `2026-09-01T00:00:00Z` |
| 994 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 995 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 996 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 997 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_workflow_step_assignee_1` | UNIQUE | `workflow_step_id, assignee_order` | UNIQUE (workflow_step_id, assignee_order) |
| `uq_workflow_step_assignee_2` | UNIQUE | `workflow_step_id, assignee_id` | UNIQUE (workflow_step_id, assignee_id) |
| `ck_workflow_step_assignee_3` | CHECK | `assignee_order` | CHECK (assignee_order >= 1) |
| `rule_workflow_step_assignee_4` | 업무 검증 | - | 주 담당자와 대체 담당자는 같을 수 없다. |
| `ck_workflow_step_assignee_status` | CHECK | `assignment_status` | CHECK (assignment_status IN ('PENDING','IN_PROGRESS','APPROVED','REJECTED','SKIPPED')) |

#### 업무 규칙

실제 처리자는 approval_action.actor_id로 기록한다. 주 담당자 또는 권한이 있는 대체 담당자 중 한 명이 같은 배정을 처리한다.

직렬은 앞선 배정 완료 후, 병렬은 현재 단계의 미완료 배정이 처리 가능하다.

[↑ 맨 위로](#top)

---

<a id="table-approval_action"></a>
### 60. 승인 처리 이력 (`approval_action`)

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
| 998 | 승인 처리 이력 ID | `approval_action_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 검토·승인 처리 이력 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 999 | Workflow 단계 ID | `workflow_step_id` | `uuid` | N | Y | `workflow_step.workflow_step_id` | Y | - | N | Y | N | Y | 처리 이력이 귀속되는 Workflow 단계. SUBMIT은 최초 비삭제 단계, CANCEL은 진행 중인 현재 단계(시작 전이면 최초 단계), REVIEW/APPROVE/REJECT는 실제 처리 단계에 연결한다. 귀속 단계가 없으면 이력을 생성하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 1000 | 처리 담당 배정 ID | `workflow_step_assignee_id` | `uuid` | N | Y | `workflow_step_assignee.assignment_id` | N | - | N | Y | N | Y | REVIEW/APPROVE/REJECT 처리 배정. SUBMIT/CANCEL은 NULL | `00000000-0000-0000-0000-000000000001` |
| 1001 | 처리 유형 | `action_type` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | 수행한 처리 유형. SUBMIT, REVIEW, APPROVE, REJECT, CANCEL | `APPROVE` |
| 1002 | 처리자 ID | `actor_id` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 실제 검토·승인·반려 처리를 수행한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 1003 | 처리 의견 | `action_comment` | `text` | N | N | - | N | - | N | N | N | Y | 검토·승인 처리 시 입력한 의견. CANCEL이면 취소 사유를 저장하며 필수 | `검토 결과 이상 없음` |
| 1004 | 반려 사유 | `rejection_reason` | `text` | N | N | - | N | - | N | N | N | Y | REJECT 처리 시 입력하는 반려 사유. REJECT에서 필수 | `증적 파일 보완 필요` |
| 1005 | 전자서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 처리에 연결된 전자서명 기록. 서명이 있으면 signer_id는 actor_id와 같고 대상 테이블·레코드·버전은 Workflow 대상과 일치해야 한다. SUBMIT/REVIEW/APPROVE/REJECT의 서명 동작은 action_type과 같아야 한다. CANCEL은 서명 없이 취소 사유와 Audit Trail을 기록한다. | `00000000-0000-0000-0000-000000000001` |
| 1006 | 처리 시각 | `acted_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 상신·검토·승인·반려가 실제 처리된 시각 | `2026-09-01T10:00:00Z` |
| 1007 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 레코드 생성 시각(UTC) | `2026-09-01T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_approval_action_1` | 업무 검증 | - | REVIEW/APPROVE/REJECT일 때 workflow_step_assignee_id는 필수이다. SUBMIT/CANCEL에서는 NULL이다. |
| `rule_approval_action_2` | 업무 검증 | - | 한 담당 배정에 유효한 REVIEW/APPROVE/REJECT 처리 이력은 한 건만 존재한다. |
| `rule_approval_action_3` | 업무 검증 | - | SUBMIT/REVIEW/APPROVE/REJECT는 전자서명이 필수이다. CANCEL은 전자서명 없이 취소 사유(action_comment)와 감사기록을 남긴다. |
| `uq_approval_assignee_action` | UNIQUE | `workflow_step_assignee_id, action_type` | UNIQUE (workflow_step_assignee_id) WHERE action_type IN ('REVIEW','APPROVE','REJECT') |
| `ck_approval_action_type` | CHECK | `action_type` | CHECK (action_type IN ('SUBMIT','REVIEW','APPROVE','REJECT','CANCEL')) |
| `ck_approval_action_reject_reason` | CHECK | `action_type, rejection_reason` | CHECK (action_type <> 'REJECT' OR (rejection_reason IS NOT NULL AND length(trim(rejection_reason)) > 0)) |
| `ck_approval_action_cancel_reason` | CHECK | `action_type, action_comment` | CHECK (action_type <> 'CANCEL' OR (action_comment IS NOT NULL AND length(trim(action_comment)) > 0)) |

#### 업무 규칙

담당 배정의 workflow_step_id와 이력의 workflow_step_id는 같아야 한다. 실제 처리자는 주 담당자 또는 대체 담당자이며 전자서명 signer_id와 일치한다.

SUBMIT은 최초 단계, CANCEL은 현재 단계(시작 전이면 최초 단계)에 기록한다. 서명 대상 테이블·PK·버전 및 동작은 workflow_instance와 action_type에 일치해야 한다.

동일 배정의 반려를 승인으로 덮어쓰지 않는다. 재상신 시 새 Workflow 또는 새 단계·담당 배정을 생성한다. 처리 대상 단계와 배정을 잠근 뒤 현재 진행 상태·미처리 여부·대체자 자격을 다시 검사하고 서명·처리 이력·단계 및 전체 상태 전환을 원자적으로 저장한다.

[↑ 맨 위로](#top)

---

<a id="table-project_workflow_config"></a>
### 61. 프로젝트 결재선 설정 (`project_workflow_config`)

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
| 1008 | 결재선 설정 ID | `config_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1009 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1010 | 수행 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | N | - | N | Y | N | Y | 활동별 설정일 때 지정. 프로젝트 기본 설정은 NULL | `00000000-0000-0000-0000-000000000001` |
| 1011 | 설정 버전 | `config_version` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 적용 후 변경할 때 증가하는 설정판 | `1` |
| 1012 | 결재선 구성 | `route_definition` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 작성자 단계(author_stage), 검토 단계 배열(review_stages), 승인 단계 배열(approval_stages). 각 단계에 순서·SERIAL/PARALLEL·주/대체 담당자 배열 저장 | - |
| 1013 | 적용 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 현재 적용 중인 설정 여부. 새 설정 적용 시 이전 행은 FALSE로 전환하고 이전 적용 서명·시각·원문은 보존한다. | `FALSE` |
| 1014 | 적용 서명 ID | `applied_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | electronic_signature.signature_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1015 | 적용 시각 | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 적용 시각 값 | `2026-09-01T00:00:00Z` |
| 1016 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 값 | `2026-09-01T00:00:00Z` |
| 1017 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1018 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 값 | `2026-09-01T00:00:00Z` |
| 1019 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

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
### 62. 공통 추적 관계 (`traceability_link`)

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
| 1020 | 추적 관계 ID | `traceability_link_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ 항목 간 추적 관계 고유 식별자. 삭제되지 않은 데이터는 (project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) 조합의 중복을 허용하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 1021 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 추적 관계가 속한 프로젝트 ID | `00000000-0000-0000-0000-000000000001` |
| 1022 | 출발 엔터티 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 연결 대상 유형: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. 각 유형의 특정 개정 행 PK를 연결한다. | `REQUIREMENT` |
| 1023 | 출발 엔터티 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 유형에 해당하는 테이블의 특정 개정 행 PK. 물리 FK가 아닌 다형 참조; 논리 key나 표시 번호로 대체하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 1024 | 연결 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 연결 대상 유형: REQUIREMENT, FDS_SPEC, DDS_SPEC, DQ_ITEM, FRA_ITEM, IQ_ITEM, OQ_ITEM, PQ_ITEM. 각 유형의 특정 개정 행 PK를 연결한다. | `OQ_ITEM` |
| 1025 | 연결 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 대상 유형에 해당하는 테이블의 특정 개정 행 PK. 물리 FK가 아닌 다형 참조; 논리 key나 표시 번호로 대체하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 1026 | 관계 유형 | `link_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | IMPLEMENTED_BY, ASSESSED_BY, VERIFIED_BY, MITIGATED_BY | `VERIFIED_BY` |
| 1027 | 연결 근거 | `link_reason` | `text` | N | N | - | N | - | N | N | N | Y | 두 산출물 항목을 연결한 업무적 근거 | `URS 요구사항을 IQ 시험으로 검증` |
| 1028 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 1029 | 작성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 생성한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 1030 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 추적 관계 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 1031 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 추적 관계를 최종 수정한 사용자 ID | `00000000-0000-0000-0000-000000000001` |
| 1032 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 추적 관계 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_traceability_link_1` | UNIQUE | `project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type` | UNIQUE(project_id, source_entity_type, source_entity_id, target_entity_type, target_entity_id, link_type) WHERE deleted_at IS NULL |
| `rule_traceability_link_2` | 업무 검증 | - | 연결 양 끝은 동일 프로젝트에 속하고 등록된 대상 유형 및 실제 개정 행으로 존재해야 한다. |
| `rule_traceability_link_3` | 업무 검증 | - | REQUIREMENT → IQ_ITEM / OQ_ITEM / PQ_ITEM의 VERIFIED_BY는 하나의 시험에 복수 URS를 연결하는 화면 관계다. |
| `rule_traceability_link_4` | 업무 검증 | - | FDS_SPEC / DDS_SPEC는 fds_spec.fds_id / dds_spec.dds_id를 의미한다. |
| `rule_traceability_link_5` | 업무 검증 | - | 대시보드 RTM은 연결 관계와 현재 유효한 원본·수행 상태를 조회한다. RTM 전용 문서·시험·승인 상태를 추가로 생성하지 않는다. |
| `rule_traceability_link_6` | 업무 검증 | - | 연결을 개정할 때 과거 승인 대상의 원본 관계를 삭제·덮어쓰지 않는다. 새 개정에 새 관계를 만들고 과거 링크를 보존한다. |
| `ck_traceability_link_source_type` | CHECK | `source_entity_type` | CHECK (source_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM')) |
| `ck_traceability_link_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FDS_SPEC','DDS_SPEC','DQ_ITEM','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM')) |
| `ck_traceability_link_type` | CHECK | `link_type` | CHECK (link_type IN ('IMPLEMENTED_BY','ASSESSED_BY','VERIFIED_BY','MITIGATED_BY')) |

#### 업무 규칙

커버리지는 URS와 시험의 연결 여부로 집계하며 시험 PASS 또는 승인완료 비율과 구분한다. FDS·DDS 문서의 직접 URS 연결은 파일 승인 화면의 선택 관계에 한해 저장한다.

[↑ 맨 위로](#top)

---

## Deviation

<a id="table-deviation"></a>
### 63. 일탈 관리 (`deviation`)

| 항목 | 정의 |
|---|---|
| 설명 | 시험 FAIL에 따른 사유·즉시 조치, 재수행, 시정·예방 조치, 완료보고 및 종료 처리 관리 |
| Primary Key | `deviation_id` |
| 주요 참조(FK) | `project_id, approved_by, created_by, updated_by, action_workflow_id, action_signature_id, completion_signature_id, closure_signature_id, current_action_round_id, current_completion_revision_id` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1033 | 일탈 ID | `deviation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 일탈 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1034 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | 일탈이 발생한 Validation 프로젝트 | `00000000-0000-0000-0000-000000000001` |
| 1035 | 발생 대상 유형 | `source_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 실패한 원본 수행의 종류: IQ_EXECUTION, OQ_EXECUTION, PQ_EXECUTION. | `OQ_EXECUTION` |
| 1036 | 발생 대상 ID | `source_entity_id` | `uuid` | N | N | - | Y | - | N | Y | N | Y | 해당 수행 테이블의 execution_id. 특정 회차·정정 결과를 가리키는 다형 참조이며 물리 FK가 아님. | `00000000-0000-0000-0000-000000000001` |
| 1037 | 일탈 번호 | `deviation_no` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | 프로젝트 내 일탈 관리 번호 | `DEV-001` |
| 1038 | 일탈 제목 | `title` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 시험 항목명과 일탈 번호를 이용한 보고서 표시 제목. 별도 필수 입력값이 아님. | `예상 결과 불일치` |
| 1039 | 일탈 사유 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 현재 조치 개정의 일탈 사유 표시 요약. 원본 실패 사유는 source_entity_id의 수행 기록에서 조회한다. | `OQ 수행 중 예상 결과와 실제 결과 불일치` |
| 1040 | 일탈 상태 | `deviation_status` | `varchar(30)` | N | N | - | Y | `'ACTION_PENDING'` | N | Y | N | Y | ACTION_PENDING(사유·조치 승인 대기), RERUN_ALLOWED(재수행 가능), COMPLETION_PENDING(완료보고 대기), CLOSED(종료). | `ACTION_PENDING` |
| 1041 | 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 완료보고 승인 또는 사유를 입력한 종료 처리의 최종 서명 시각. | `2026-09-03T17:00:00Z` |
| 1042 | 승인자 ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 완료보고 승인 또는 종료 처리의 최종 서명 사용자. | `00000000-0000-0000-0000-000000000001` |
| 1043 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 생성 시각(UTC) | `2026-09-01T10:00:00Z` |
| 1044 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 등록 사용자 | `00000000-0000-0000-0000-000000000001` |
| 1045 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 일탈 최종 수정 시각(UTC) | `2026-09-01T10:00:00Z` |
| 1046 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 일탈 최종 수정 사용자 | `00000000-0000-0000-0000-000000000001` |
| 1047 | 삭제시각 | `deleted_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 일탈 소프트 삭제 시각 | `2026-09-02T10:00:00Z` |
| 1048 | 즉시 조치 | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | 현재 조치 개정의 즉시 조치 표시 요약. 회차별 승인 원문은 deviation_action_round에 보존한다. | - |
| 1049 | 사유·조치 승인 상태 | `action_approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | 현재 조치 개정의 승인 상태 요약. DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. 독립 편집하지 않는다. | `DRAFT` |
| 1050 | 사유·조치 승인 워크플로우 | `action_workflow_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | current_action_round_id의 workflow_instance_id에서 동기화한 현재 승인 절차. 이전 회차는 deviation_action_round에서 조회하며 독립 편집하지 않는다. | `00000000-0000-0000-0000-000000000001` |
| 1051 | 사유·조치 최종 서명 | `action_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 현재 조치 회차의 signature_id에서 동기화한 최종 승인 서명. 과거 서명은 회차별로 보존한다. | `00000000-0000-0000-0000-000000000001` |
| 1052 | 최근 재수행 ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | 최근 재수행의 유효한 결과 개정 PK. 회차의 최초 재수행과 같은 논리 시험·attempt_no에 속해야 한다. 재수행의 최초 수행 행이 중단(ABORTED)되면 연결을 해제하고, 결과 정정이 RECORDED로 확정되면 정정 행으로 갱신한다. 작성 중 정정의 취소는 영향을 주지 않는다. source_entity_type으로 테이블을 식별한다. | `00000000-0000-0000-0000-000000000001` |
| 1053 | 시정·예방 조치 | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 완료보고 개정(deviation_completion_revision)에서 동기화한 시정·예방 조치. 작성·수정은 개정 테이블에서 하며 독립 편집하지 않는다. | - |
| 1054 | 완료 보고 | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 완료보고 개정에서 동기화한 완료 보고. 작성·수정은 개정 테이블에서 하며 독립 편집하지 않는다. | - |
| 1055 | 완료보고 승인 서명 | `completion_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 승인된 완료보고 개정의 최종 승인 서명(signature_id)에서 동기화. | `00000000-0000-0000-0000-000000000001` |
| 1056 | 종료 처리 사유 | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | 승인된 미수행 종료 개정(closure_kind=CLOSE)에서 동기화한 종료 사유. 작성·수정은 개정 테이블에서 하며 독립 편집하지 않는다. | - |
| 1057 | 종료 처리 서명 | `closure_signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 승인된 미수행 종료 개정의 최종 승인 서명에서 동기화. 완료보고 승인과 구분. | `00000000-0000-0000-0000-000000000001` |
| 1058 | 현재 일탈 조치 개정 | `current_action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | N | - | N | Y | N | Y | 이 일탈의 마지막 조치 회차에서 최신 원문 개정. 최초 실패와 첫 조치 초안을 생성하는 트랜잭션 종료 시 필수 | - |
| 1059 | 현재 종결 보고 개정 | `current_completion_revision_id` | `uuid` | N | Y | `deviation_completion_revision.completion_revision_id` | N | - | N | Y | N | Y | 이 일탈의 최신 종결 보고 개정(is_current_revision=TRUE). 종결 보고를 작성하기 전에는 NULL | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deviation_1` | UNIQUE | `project_id, deviation_no` | UNIQUE(project_id, deviation_no) |
| `uq_deviation_source` | UNIQUE | `source_entity_type, source_entity_id` | UNIQUE (source_entity_type, source_entity_id) WHERE deleted_at IS NULL |
| `rule_deviation_8` | 업무 검증 | `source_entity_type, source_entity_id, deviation_status` | 같은 논리 시험(item_key)에 종료되지 않은(deviation_status<>'CLOSED') deviation은 최대 1건이다. 생성 시 논리 시험 단위로 잠근 뒤 검사한다. |
| `ck_deviation_action_approval_status` | CHECK | `action_approval_status` | CHECK (action_approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `rule_deviation_1` | 업무 검증 | `project_id, source_entity_id, deviation_no` | source_entity_id 및 rerun_execution_id는 동일 프로젝트·동일 논리 시험에 속해야 한다 |
| `rule_deviation_2` | 업무 검증 | - | RERUN_ALLOWED 전환에는 사유·조치 APPROVED와 action_signature_id가 필요하다. |
| `rule_deviation_3` | 업무 검증 | `source_entity_id, rerun_execution_id, current_action_round_id` | 일탈 원본 source_entity_id는 최초 실패 기록을 유지한다. 승인된 조치에 따른 재수행이 FAIL이면 새 일탈을 만들지 않고 deviation.rerun_execution_id에 그 재수행의 유효 결과 개정을 기록하며, 다음 round_number의 deviation_action_round(failed_execution_id = 그 재수행 execution_id)를 생성하고 current_action_round_id를 새 회차로 갱신한다. |
| `rule_deviation_4` | 업무 검증 | `corrective_action, completion_report, completion_signature_id` | COMPLETION_PENDING 이후 완료보고 개정(closure_kind=COMPLETION)의 최종 승인으로 종료하며, 승인본에서 동기화한 corrective_action / completion_report / completion_signature_id가 필수. 완료보고 승인 직전 최근 재수행의 유효 결과가 PASS이고 그 결과 개정의 record_status가 APPROVED인지 검사한다. |
| `rule_deviation_5` | 업무 검증 | `closure_reason, closure_signature_id` | 재수행 없이 종료할 때 RERUN_ALLOWED 상태이고 현재 조치 회차에 연결된 재수행(rerun_execution_id)이 없어야 하며, 미수행 종료 개정(closure_kind=CLOSE)의 최종 승인으로 종료하고 승인본에서 동기화한 closure_reason / closure_signature_id가 필수. 두 종료 경로는 구분하고 과거 서명·결과는 삭제하지 않는다. |
| `rule_deviation_6` | 업무 검증 | `approved_by, completion_signature_id, closure_signature_id` | CLOSED이면 completion_signature_id / closure_signature_id 중 정확히 하나와 approved_by / approved_at이 필수이며, 최종 서명자·서명 시각과 일치해야 한다. |
| `rule_deviation_7` | 업무 검증 | - | 재수행 판정은 rerun_execution_id의 qualification_result에서 조회한다. 원본 실패 사유·즉시 조치는 수행 기록에서 당시 값을 재현할 수 있다. |
| `rule_deviation_9` | 업무 검증 | `deviation_status` | deviation_status는 다음 전이만 허용한다. ACTION_PENDING→RERUN_ALLOWED: 현재 조치 회차 최종 승인. RERUN_ALLOWED→ACTION_PENDING: 재수행 결과 FAIL 등록(다음 조치 회차 생성과 같은 트랜잭션). RERUN_ALLOWED→COMPLETION_PENDING: 재수행 결과 PASS 등록. RERUN_ALLOWED→CLOSED: 재수행 없이 종료 승인(rule_deviation_5). COMPLETION_PENDING→CLOSED: 완료보고 승인(rule_deviation_4). COMPLETION_PENDING→ACTION_PENDING: 완료보고 결재가 진행 중이 아닐 때 재수행 결과를 FAIL로 정정(다음 조치 회차 생성과 같은 트랜잭션). 재수행 중단(최초 수행 행 ABORTED)은 상태를 바꾸지 않으며(RERUN_ALLOWED 유지), 작성 중인 결과 정정의 취소도 상태를 바꾸지 않는다. CLOSED 이후와 그 외 전이는 허용하지 않는다. |
| `rule_deviation_10` | 업무 검증 | `corrective_action, completion_report, closure_reason, current_completion_revision_id` | 완료보고·미수행 종료 원문은 deviation_completion_revision에서 작성·개정하며 이 테이블의 해당 컬럼은 승인본에서 동기화한 요약으로 직접 편집하지 않는다. 반려·취소 후 수정은 새 종결 보고 개정으로 처리한다. |
| `ck_deviation_source_type` | CHECK | `source_entity_type` | CHECK (source_entity_type IN ('IQ_EXECUTION','OQ_EXECUTION','PQ_EXECUTION')) |
| `ck_deviation_status` | CHECK | `deviation_status` | CHECK (deviation_status IN ('ACTION_PENDING','RERUN_ALLOWED','COMPLETION_PENDING','CLOSED')) |

#### 업무 규칙

종결 처리자·시각은 최종 전자서명 값과 일치시킨다.

최초 FAIL 등록과 첫 조치 회차 생성은 한 트랜잭션으로 처리한다. current_action_round_id는 이 일탈에 속하며 마지막 round_number의 is_current_revision=TRUE인 행이어야 한다. 현재 사유·조치·승인 상태·Workflow·서명은 해당 행에서 동기화하고 과거 원문은 회차 행에서 조회한다. 최근 재수행이 FAIL이면 기존 일탈을 유지하면서 다음 조치 회차를 생성하고 deviation_status를 ACTION_PENDING으로 되돌린다. 재수행이 PASS이면 COMPLETION_PENDING으로 전환한다.

완료보고·미수행 종료의 결재와 서명은 deviation_completion_revision의 PK를 대상으로 하며 전자서명의 버전 규칙을 따른다. signed_payload에는 기준 조치 회차, 최초 실패 PK, 최근 재수행의 정확한 결과 개정 PK, 완료보고 또는 종료 사유를 고정한다. 완료보고 종료에는 최근 재수행이 PASS이어야 한다. CLOSED 이후 사유·조치·완료보고·서명·결과 참조를 수정하지 않는다.

[↑ 맨 위로](#top)

---

<a id="table-deviation_action_round"></a>
### 64. 일탈 조치 회차 (`deviation_action_round`)

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
| 1060 | 일탈 조치 개정 ID | `action_round_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 하나의 조치 회차에서 작성한 특정 원문 개정 식별자 | - |
| 1061 | 일탈 ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | 일탈 원본 | - |
| 1062 | 조치 회차 | `round_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 최초 실패는 1. 승인 후 재수행이 실패하면 다음 회차를 생성한다. | `1` |
| 1063 | 회차 내 원문 개정 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 같은 실패에 대한 반려 후 수정·재상신 시 증가한다. | `1` |
| 1064 | 회차 최신 개정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 같은 일탈·조치 회차에서 최신 작성 개정은 한 건 | - |
| 1065 | 해당 회차 실패 수행 ID | `failed_execution_id` | `uuid` | N | N | - | Y | - | N | N | N | Y | deviation.source_entity_type에 해당하는 수행 PK. 첫 회차는 최초 실패, 다음 회차는 직전 회차의 실패한 재수행. 다형 참조이며 물리 FK 아님 | - |
| 1066 | 회차별 일탈 사유 | `description` | `text` | N | N | - | Y | - | N | N | N | Y | 해당 실패에 대한 승인 대상 사유 원문 | - |
| 1067 | 회차별 즉시 조치 | `immediate_action` | `text` | N | N | - | Y | - | N | N | N | Y | 해당 실패에 대해 수행한 조치 원문 | - |
| 1068 | 조치 승인 상태 | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. REJECTED·CANCELLED는 이 원문 개정의 종결 상태 | - |
| 1069 | 조치 승인 절차 | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 action_round_id와 ACTION-{round_number}-{revision_number}를 대상으로 하는 DEVIATION_ACTION 결재 | - |
| 1070 | 조치 최종 승인 서명 | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 해당 조치 원문 개정의 최종 승인 서명 | - |
| 1071 | 승인으로 허용한 재수행 ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | N | N | Y | 이 승인으로 시작한 재수행 PK. source_entity_type으로 테이블을 식별한다. 재수행하지 않고 종료하거나 재수행을 중단(ABORTED)해 연결을 해제한 경우 NULL | - |
| 1072 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 원문 개정 생성 시각 | - |
| 1073 | 작성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 조치 원문 작성자 | - |
| 1074 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 초안 편집 또는 관리 상태 변경 시각 | - |
| 1075 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 처리자 | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deviation_action_revision` | UNIQUE | `deviation_id, round_number, revision_number` | UNIQUE (deviation_id, round_number, revision_number) |
| `uq_deviation_action_current` | UNIQUE | `deviation_id, round_number, is_current_revision` | UNIQUE (deviation_id, round_number) WHERE is_current_revision=TRUE |
| `uq_deviation_action_approved` | UNIQUE | `deviation_id, round_number, approval_status` | UNIQUE (deviation_id, round_number) WHERE approval_status='APPROVED' |
| `ck_deviation_action_sequence` | CHECK | `round_number, revision_number` | CHECK (round_number >= 1 AND revision_number >= 1) |
| `ck_deviation_action_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deviation_action_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_action_rerun` | CHECK | `rerun_execution_id, approval_status` | CHECK (rerun_execution_id IS NULL OR approval_status='APPROVED') |
| `rule_deviation_action_chain` | 업무 검증 | `failed_execution_id, rerun_execution_id` | 실패·재수행은 일탈과 같은 프로젝트·단계·논리 시험에 속한다. 첫 실패는 일탈 source_entity_id, 다음 회차 실패는 직전 승인 회차의 재수행 또는 그 재수행과 동일 프로토콜·attempt_no에 속하는 정정 결과 개정이며 FAIL이어야 한다. 같은 회차의 개정은 failed_execution_id를 유지한다. |

#### 업무 규칙

상신 후 조치 원문을 잠근다. 반려·취소 후 내용을 바꿀 때는 같은 round_number의 새 revision_number와 PK를 생성하고 이전 행·Workflow·서명은 보존한다. 승인된 회차는 새 원문 개정을 만들지 않는다. 재수행 FAIL은 다음 round_number로 연결한다.

승인 Workflow와 서명은 deviation이 아닌 이 테이블의 정확한 action_round_id·ACTION-{round_number}-{revision_number}를 가리킨다. 사유·조치 원문과 failed_execution_id는 서명 원문에 포함한다. 승인 이후 조치 원문·실패 참조·승인 상태·최종 서명은 변경하지 않는다. 이후 최초 재수행 연결과 수정자·수정 시각 등 관리 메타데이터만 갱신할 수 있다. rerun_execution_id는 최초 연결 후 다른 회차로 덮어쓰지 않는다. 단, 연결된 재수행의 최초 수행 행(record_revision=1)이 ABORTED이면 연결을 해제(NULL)하고 새 재수행을 연결할 수 있으며(결과 정정 행의 취소는 연결에 영향을 주지 않는다), 해제 전 값과 중단된 수행 행은 Audit Trail과 해당 수행 테이블에 보존한다.

REVIEW·APPROVAL·APPROVED에는 workflow_instance_id가 필수이며 승인 Workflow의 대상·범위·버전, 최종 서명과 서명 원문은 이 조치 개정과 일치해야 한다.

재수행 생성은 일탈과 승인 회차를 잠그고 deviation_status=RERUN_ALLOWED·승인 상태·현재 회차·기존 재수행 부재를 검사한 뒤 시험 수행 생성, 회차 연결, 일탈 요약 갱신을 한 트랜잭션으로 처리한다. 수행 정정은 새 수행 PK를 사용하되 원래 재수행 연결을 유지하고 동일 attempt_no의 record_revision으로 정정 계보를 조회한다. 다음 실패의 판단과 완료보고는 해당 시점의 유효한 결과 개정을 명시적으로 식별한다.

[↑ 맨 위로](#top)

---

<a id="table-deviation_completion_revision"></a>
### 65. 일탈 종결 보고 개정 (`deviation_completion_revision`)

| 항목 | 정의 |
|---|---|
| 설명 | 일탈 완료보고(시정·예방 조치, 완료 보고)와 재수행 없는 종료 사유의 원문 개정·승인 |
| Primary Key | `completion_revision_id` |
| 주요 참조(FK) | `deviation_id, action_round_id, workflow_instance_id, signature_id, created_by, updated_by` |
| GxP 중요도 | Critical |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존. 상신·승인한 원문과 반려·취소된 개정은 삭제하지 않는다. |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1076 | 종결 보고 개정 ID | `completion_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 종결 보고의 특정 원문 개정 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1077 | 일탈 ID | `deviation_id` | `uuid` | N | Y | `deviation.deviation_id` | Y | - | N | Y | N | Y | 종결 대상 일탈 | `00000000-0000-0000-0000-000000000001` |
| 1078 | 종결 구분 | `closure_kind` | `varchar(20)` | N | N | - | Y | - | N | Y | N | Y | COMPLETION=재수행 성공 후 완료보고, CLOSE=재수행 없이 종료 | `COMPLETION` |
| 1079 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | `1` | N | N | N | Y | 일탈별 1부터 증가. 반려·취소 후 수정하면 다음 순번의 새 행을 만든다 | `1` |
| 1080 | 최신 개정 여부 | `is_current_revision` | `boolean` | N | N | - | Y | `TRUE` | N | Y | N | Y | 같은 일탈의 최신 종결 보고 개정 여부 | `TRUE` |
| 1081 | 기준 조치 회차 | `action_round_id` | `uuid` | N | Y | `deviation_action_round.action_round_id` | Y | - | N | Y | N | Y | 작성 시점의 현재 조치 회차(deviation.current_action_round_id) | `00000000-0000-0000-0000-000000000001` |
| 1082 | 근거 재수행 결과 ID | `rerun_execution_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | COMPLETION의 근거인 재수행 유효 결과 개정 PK. deviation.source_entity_type으로 테이블을 식별하는 다형 참조이며 물리 FK가 아니다. CLOSE는 NULL | `00000000-0000-0000-0000-000000000001` |
| 1083 | 시정·예방 조치 | `corrective_action` | `text` | N | N | - | N | - | N | N | N | Y | COMPLETION의 시정·예방 조치 원문. 상신 시 필수 | `같은 모델 장비 3대 점검, 설치 체크리스트에 경보 기준값 항목 추가` |
| 1084 | 완료 보고 | `completion_report` | `text` | N | N | - | N | - | N | N | N | Y | COMPLETION의 완료 보고 원문. 상신 시 필수 | `경보 기준값 설정 오류. 8도로 수정 후 재시험 성공` |
| 1085 | 종료 사유 | `closure_reason` | `text` | N | N | - | N | - | N | N | N | Y | CLOSE의 재수행 없이 종료하는 사유 원문. 상신 시 필수 | - |
| 1086 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 반려·취소 후 다시 작성한 사유. revision_number ≥ 2이면 필수 | `QA 반려 의견 반영` |
| 1087 | 승인 상태 | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | Y | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. REJECTED·CANCELLED는 이 개정의 종결 상태 | `DRAFT` |
| 1088 | 승인 절차 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | 이 개정을 대상으로 하는 DEVIATION_COMPLETION 또는 DEVIATION_CLOSE 결재 | `00000000-0000-0000-0000-000000000001` |
| 1089 | 최종 승인 서명 ID | `signature_id` | `uuid` | N | Y | `electronic_signature.signature_id` | N | - | N | Y | N | Y | 이 개정의 최종 승인 전자서명 | `00000000-0000-0000-0000-000000000001` |
| 1090 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 원문 개정 생성 시각 | `2026-09-03T10:00:00Z` |
| 1091 | 작성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 종결 보고 작성자 | `00000000-0000-0000-0000-000000000001` |
| 1092 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 초안 편집 또는 관리 상태 변경 시각 | `2026-09-03T10:00:00Z` |
| 1093 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 처리자 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deviation_completion_revision` | UNIQUE | `deviation_id, revision_number` | UNIQUE (deviation_id, revision_number) |
| `uq_deviation_completion_current` | UNIQUE | `deviation_id, is_current_revision` | UNIQUE (deviation_id) WHERE is_current_revision=TRUE |
| `uq_deviation_completion_approved` | UNIQUE | `deviation_id, approval_status` | UNIQUE (deviation_id) WHERE approval_status='APPROVED' |
| `ck_deviation_completion_sequence` | CHECK | `revision_number` | CHECK (revision_number >= 1) |
| `ck_deviation_completion_kind` | CHECK | `closure_kind` | CHECK (closure_kind IN ('COMPLETION','CLOSE')) |
| `ck_deviation_completion_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deviation_completion_content` | CHECK | `closure_kind, corrective_action, completion_report, rerun_execution_id, closure_reason` | CHECK ((closure_kind='COMPLETION' AND closure_reason IS NULL) OR (closure_kind='CLOSE' AND corrective_action IS NULL AND completion_report IS NULL AND rerun_execution_id IS NULL)) |
| `ck_deviation_completion_approval` | CHECK | `approval_status, workflow_instance_id, signature_id` | CHECK (approval_status <> 'APPROVED' OR (workflow_instance_id IS NOT NULL AND signature_id IS NOT NULL)) |
| `ck_deviation_completion_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |
| `rule_deviation_completion_submit` | 업무 검증 | `closure_kind, approval_status` | 상신 시 action_round_id는 deviation.current_action_round_id와 같아야 한다. COMPLETION은 deviation_status=COMPLETION_PENDING이고 corrective_action·completion_report·rerun_execution_id가 필수이며 rerun_execution_id는 최근 재수행의 유효 결과 개정이어야 한다. CLOSE는 deviation_status=RERUN_ALLOWED이고 현재 조치 회차에 연결된 재수행이 없어야 하며 closure_reason이 필수다. |
| `rule_deviation_completion_lock` | 업무 검증 | `approval_status` | 상신(REVIEW) 이후 원문을 잠근다. 반려·취소 후 수정은 같은 일탈의 다음 revision_number로 새 행을 만들고 이전 행·Workflow·서명은 보존한다. 같은 일탈에 진행 중(DRAFT·REVIEW·APPROVAL)인 개정은 최대 1개다. |
| `rule_deviation_completion_sync` | 업무 검증 | `approval_status` | 최종 승인 트랜잭션에서 deviation의 corrective_action·completion_report 또는 closure_reason, completion_signature_id 또는 closure_signature_id, approved_by·approved_at을 이 개정에서 동기화하고 deviation_status를 CLOSED로 전환한다(rule_deviation_9). |
| `rule_deviation_completion_reopen` | 업무 검증 | `approval_status` | deviation_status가 ACTION_PENDING으로 돌아가면(rule_deviation_9) 상신 전 DRAFT 개정은 CANCELLED로 종결한다. |

#### 업무 규칙

완료보고와 재수행 없는 종료는 일탈 조치(deviation_action_round)와 같은 방식으로 원문 개정을 관리한다. 화면에서는 반려된 보고를 "수정"하는 것으로 보이지만 DB에는 새 개정 행이 생성되며, 1차·2차 원문을 이 테이블에서 바로 비교할 수 있다.

결재와 전자서명은 deviation이 아닌 이 테이블의 completion_revision_id를 대상으로 하며 target_version은 COMPLETE-{조치 회차}-{revision_number} 또는 CLOSE-{조치 회차}-{revision_number}이다. signed_payload에는 기준 조치 회차, 최초 실패 PK, 근거 재수행 결과 개정 PK, 시정·예방 조치와 완료 보고 또는 종료 사유를 고정한다.

[↑ 맨 위로](#top)

---

## Document

<a id="table-deliverable_document"></a>
### 66. 산출물 문서 (`deliverable_document`)

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
| 1094 | 문서 ID | `document_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1095 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | Y | - | N | Y | N | Y | validation_project.project_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1096 | 수행 활동 ID | `project_activity_id` | `uuid` | N | Y | `project_activity.project_activity_id` | Y | - | N | Y | N | Y | project_activity.project_activity_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1097 | 문서 구분 | `document_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | STAGE_DELIVERABLE / PROTOCOL / RECORD / SUMMARY | `RECORD` |
| 1098 | 문서 번호 | `document_number` | `varchar(100)` | N | N | - | N | - | N | N | N | Y | 산출물 화면의 문서 번호. 승인 전 NULL 허용 | `OQ-R-001` |
| 1099 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1100 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1101 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1102 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_deliverable_document_1` | 업무 검증 | - | 문서의 project_activity_id는 동일 project_id에 속해야 한다. RTM은 독립 수행 활동/필수 산출물에 포함하지 않는다. |
| `rule_deliverable_document_2` | 업무 검증 | - | 문서번호가 있는 경우 프로젝트 안에서 유일하게 관리한다. 시험 프로토콜과 결과 문서는 문서 구분으로 나눈다. |
| `ck_deliverable_document_type` | CHECK | `document_type` | CHECK (document_type IN ('STAGE_DELIVERABLE','PROTOCOL','RECORD','SUMMARY')) |

[↑ 맨 위로](#top)

---

<a id="table-deliverable_revision"></a>
### 67. 산출물 개정 (`deliverable_revision`)

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
| 1103 | 문서 개정 ID | `document_revision_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1104 | 문서 ID | `document_id` | `uuid` | N | Y | `deliverable_document.document_id` | Y | - | N | Y | N | Y | deliverable_document.document_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1105 | 문서 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 문서 제목 | `운전 적격성 평가 결과서` |
| 1106 | 문서 표시 버전 | `version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 문서 표시 버전 | `v1.0` |
| 1107 | 개정 순번 | `revision_number` | `integer` | N | N | - | Y | - | N | N | N | Y | 개정 순번 | `1` |
| 1108 | 개정 사유 | `revision_reason` | `text` | N | N | - | N | - | N | N | N | Y | 개정 사유 | `시험 결과 반영` |
| 1109 | 최신 작성본 여부 | `is_current_version` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 최신 작성본 여부 | `TRUE` |
| 1110 | 문서 승인 상태 | `approval_status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / REVIEW / APPROVAL / APPROVED / REJECTED / CANCELLED. REJECTED·CANCELLED 후 수정은 새 문서 개정으로 처리 | `DRAFT` |
| 1111 | 문서 결재 ID | `workflow_instance_id` | `uuid` | N | Y | `workflow_instance.workflow_instance_id` | N | - | N | Y | N | Y | workflow_instance.workflow_instance_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1112 | 근거 업무 승인 버전 | `source_version` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 화면의 승인 데이터 버전. 문서 자체 버전과 구분 | `v1.0` |
| 1113 | 근거 업무 개정 목록 | `source_refs` | `jsonb` | N | N | - | Y | - | N | N | N | Y | [{table_name,record_id,version}] 배열. 여러 업무 항목의 정확한 개정과 승인된 수행 회차를 고정 | `[{"table_name":"oq_execution","record_id":"00000000-0000-0000-0000-000000000001","version":"1-1"}]` |
| 1114 | 승인 데이터 표 | `approved_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 화면의 승인 데이터 열/행. {columns:string[],rows:string[][]} | `{"columns":["항목","결과"],"rows":[["OQ-001","Pass"]]}` |
| 1115 | Part 11 표 | `part11_table_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | QIA 산출물의 Part 11 문항·응답·판정 표. 다른 활동은 NULL | - |
| 1116 | 개정 이력 표 | `revision_table_snapshot` | `jsonb` | N | N | - | Y | - | N | N | N | Y | 작성 당시 개정 이력 열/행 | `{"columns":["버전","사유"],"rows":[["v1.0","최초 작성"]]}` |
| 1117 | 재승인 원인 활동 | `reapproval_source_stage` | `varchar(20)` | N | N | - | N | - | N | N | N | Y | 상위 활동 변경으로 재승인이 필요한 경우의 활동 코드. RTM 제외 | `URS` |
| 1118 | 재승인 사유 | `reapproval_reason` | `text` | N | N | - | N | - | N | N | N | Y | 재승인 사유 | `URS 개정으로 내용 재검토` |
| 1119 | 승인 유효성 상실 시각 | `invalidated_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 승인 유효성 상실 시각 | `2026-09-01T00:00:00Z` |
| 1120 | 최종 승인자 ID | `approved_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1121 | 최종 승인 시각 | `approved_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 최종 승인 시각 | `2026-09-01T00:00:00Z` |
| 1122 | PDF 파일 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 생성 PDF를 보관한 경우 연결 | `00000000-0000-0000-0000-000000000001` |
| 1123 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1124 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1125 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1126 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1127 | 문서 검증 대상 기준 | `system_baseline_id` | `uuid` | N | Y | `project_system_baseline.baseline_id` | N | - | N | Y | N | Y | 문서 상신 시 고정한 동일 프로젝트의 기준. 상신·승인 시 필수. 이전 수행은 각 수행의 기준을 함께 표시한다. | - |
| 1128 | 평가 질문·판정 규칙 원문 | `evaluation_definition_snapshot` | `jsonb` | N | N | - | N | - | N | N | N | Y | QIA/FRA 문서의 근거 평가별 질문·규칙 버전과 실제 정의, 입력·결과를 고정한 배열. 평가와 무관한 문서는 NULL | - |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deliverable_revision_1` | UNIQUE | `document_id, revision_number` | UNIQUE(document_id,revision_number) |
| `uq_deliverable_revision_1_2` | UNIQUE | `document_id, version` | UNIQUE(document_id,version) |
| `rule_deliverable_revision_1` | 업무 검증 | `document_id, version, revision_number, is_current_version` | 문서별 is_current_version=TRUE는 최대 1개 |
| `rule_deliverable_revision_2` | 업무 검증 | - | 문서 승인과 근거 업무 항목 승인은 별도이다. source_refs는 여러 원본 항목의 정확한 개정·프로젝트·승인 여부를 검증하고 당시 표와 함께 보존한다. |
| `rule_deliverable_revision_3` | 업무 검증 | - | APPROVED이면 문서 결재 이력·최종 승인자·시각이 필요하다. 승인된 본문/표/근거 연결은 변경하지 않고 새 개정을 작성한다. |
| `rule_deliverable_revision_4` | 업무 검증 | - | 이후 원본이 바뀌어도 과거 승인본을 삭제하지 않는다. 현재 유효성은 invalidated_at과 재승인 사유로 별도 판단한다. |
| `uq_document_current_revision` | UNIQUE | `document_id` | UNIQUE (document_id) WHERE is_current_version=TRUE |
| `ck_deliverable_revision_status` | CHECK | `approval_status` | CHECK (approval_status IN ('DRAFT','REVIEW','APPROVAL','APPROVED','REJECTED','CANCELLED')) |
| `ck_deliverable_revision_revision_reason` | CHECK | `revision_number, revision_reason` | CHECK (revision_number = 1 OR (revision_reason IS NOT NULL AND length(trim(revision_reason)) > 0)) |

#### 업무 규칙

PDF 생성 상태는 문서 승인 상태를 대신하지 않는다. VP 원본 목차는 vp_section이고, VP 산출물은 승인된 원본을 반영하므로 같은 내용을 두 곳에서 독립 편집하지 않는다.

IQ/OQ/PQ의 source_refs에는 해당 평가 개정과 문서에 포함된 시험 항목·수행의 정확한 PK·버전을 모두 기록한다. 프로토콜 문서는 평가 item_revision_refs의 전체 구성을 고정하며 결과 문서는 그 구성에 속한 수행의 정확한 회차·결과 개정을 연결한다. 문서만 개정할 때 같은 평가·시험 참조를 그대로 재사용할 수 있다. system_baseline_id와 참조 대상은 같은 프로젝트여야 한다. 다른 기준에서 수행한 결과를 포함하면 해당 수행의 기준과 재사용 판단 근거를 본문에 명시한다.

문서 상신·승인 시 system_baseline_id는 필수이며, QIA/FRA 문서는 evaluation_definition_snapshot도 필수이다. 해당 사본은 입력과 버전별 실제 정의로 결과를 재계산할 수 있어야 하며 단순 결과 표만 저장하지 않는다. 상신 후 본문·구성·질문 및 규칙 사본은 잠그고 반려·취소 후 수정은 새 문서 개정으로 처리한다. 생성 PDF는 파일 생성 이후 연결하되 이미 서명한 업무 원문을 바꾸지 않으며, PDF 파일의 원본 동일성은 file_asset의 해시로 별도 검증한다.

[↑ 맨 위로](#top)

---

<a id="table-deliverable_section"></a>
### 68. 산출물 목차/본문 (`deliverable_section`)

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
| 1129 | 섹션 ID | `section_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1130 | 문서 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | Y | - | N | Y | N | Y | deliverable_revision.document_revision_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1131 | 섹션 키 | `section_key` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 섹션 키 | `purpose` |
| 1132 | 목차 제목 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 목차 제목 | `목적` |
| 1133 | 본문 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | 본문 | `본 시험의 목적을 기술한다.` |
| 1134 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1135 | 작성 출처 | `source_type` | `varchar(20)` | N | N | - | Y | - | N | N | N | Y | MANUAL / AI / APPROVED / HISTORY / PART11 | `MANUAL` |
| 1136 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1137 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1138 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1139 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_deliverable_section_1` | UNIQUE | `document_revision_id, section_key` | UNIQUE(document_revision_id,section_key) |
| `uq_deliverable_section_1_2` | UNIQUE | `document_revision_id, sort_order` | UNIQUE(document_revision_id,sort_order) |
| `ck_deliverable_section_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_deliverable_section_2` | 업무 검증 | - | MANUAL/AI 본문은 상위 문서 개정이 DRAFT일 때만 편집 가능하며 상신(REVIEW) 이후에는 잠근다. 반려·취소 후 수정은 새 문서 개정에서 한다. APPROVED/HISTORY/PART11 섹션은 해당 스냅샷에서 표시한다. 승인된 문서 섹션은 직접 수정·삭제하지 않는다. |
| `ck_deliverable_section_source_type` | CHECK | `source_type` | CHECK (source_type IN ('MANUAL','AI','APPROVED','HISTORY','PART11')) |

[↑ 맨 위로](#top)

---

## Report

<a id="table-report_generation"></a>
### 69. 리포트 생성 작업 (`report_generation`)

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
| 1140 | 리포트 생성 ID | `report_generation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 리포트 생성 작업 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1141 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 단위 리포트인 경우 연결하며 전체 시스템 또는 조직 단위 리포트는 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 1142 | 리포트 유형 | `report_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | DELIVERABLE / AUDIT_TRAIL. 화면의 산출물 또는 감사 내보내기 | `AUDIT_TRAIL` |
| 1143 | 리포트명 | `report_name` | `varchar(200)` | N | N | - | Y | - | N | N | N | Y | 사용자에게 표시되는 생성 리포트명 | `2026년 8월 Audit Trail 리포트` |
| 1144 | 조회 시작일시 | `period_from` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 시작일시. 조회기간이 없는 리포트는 NULL 허용 | `2026-08-01T00:00:00Z` |
| 1145 | 조회 종료일시 | `period_to` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 리포트 원천 데이터 조회 종료일시. period_from보다 빠를 수 없음 | `2026-08-31T23:59:59Z` |
| 1146 | 조회 조건 | `report_parameters` | `jsonb` | N | N | - | N | - | N | N | Y | Y | 화면에서 선택한 기간·사용자·역할·메뉴 등 내보내기 필터 | `{"action_types":["CREATE","UPDATE"]}` |
| 1147 | 출력 형식 | `output_format` | `varchar(20)` | N | N | - | Y | `'PDF'` | N | Y | N | Y | 출력 형식 PDF | `PDF` |
| 1148 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1149 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | PDF/내보내기를 요청한 사용자 | `00000000-0000-0000-0000-000000000001` |
| 1150 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 사용자가 내보내기를 요청한 시각 | `2026-09-02T15:00:00Z` |
| 1151 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 생성이 시작된 시각 | `2026-09-02T15:00:05Z` |
| 1152 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 생성 성공 또는 실패가 확정된 시각 | `2026-09-02T15:01:30Z` |
| 1153 | 결과 파일 ID | `result_file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | 결과를 파일 자산으로 보관한 경우 연결. 다운로드만 수행하면 NULL 가능 | `00000000-0000-0000-0000-000000000001` |
| 1154 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인을 분류하는 시스템 오류 코드 | `REPORT_FILE_CREATE_FAILED` |
| 1155 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 리포트 생성 실패 상세 내용. 비밀번호, 토큰 등 민감정보는 저장하지 않음 | `결과 파일 저장 중 오류 발생` |
| 1156 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 레코드 생성 시각(UTC) | `2026-09-02T15:00:00Z` |
| 1157 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 내보내기를 요청한 사용자. 자동·배치 생성은 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1158 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 리포트 생성 작업 최종 수정 시각(UTC) | `2026-09-02T15:01:30Z` |
| 1159 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 내보내기 처리 결과의 수정 사용자. 자동 상태 반영은 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1160 | 산출물 개정 ID | `document_revision_id` | `uuid` | N | Y | `deliverable_revision.document_revision_id` | N | - | N | Y | N | Y | deliverable_revision.document_revision_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_report_generation_1` | 업무 검증 | `document_revision_id` | DELIVERABLE이면 document_revision_id 필수이며 해당 프로젝트와 일치해야 한다. AUDIT_TRAIL이면 기간/필터로 조회 범위를 고정한다. |
| `rule_report_generation_2` | 업무 검증 | `period_from, period_to` | period_from과 period_to가 모두 있으면 period_from<=period_to. generation_status에는 자동 재시도 대기 상태를 두지 않는다. |
| `ck_report_generation_type` | CHECK | `report_type` | CHECK (report_type IN ('DELIVERABLE','AUDIT_TRAIL')) |
| `ck_report_generation_status` | CHECK | `generation_status` | CHECK (generation_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |

#### 업무 규칙

본문·목차·버전·문서 승인 상태는 deliverable_revision과 deliverable_section에 저장한다. 이 테이블은 내보내기 결과만 관리한다.

[↑ 맨 위로](#top)

---

## AI

<a id="table-ai_generation_job"></a>
### 70. AI 생성 작업 (`ai_generation_job`)

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
| 1161 | AI 작업 ID | `ai_job_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 생성 작업 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1162 | 프로젝트 ID | `project_id` | `uuid` | N | Y | `validation_project.project_id` | N | - | N | Y | N | Y | 프로젝트 연결 | `00000000-0000-0000-0000-000000000001` |
| 1163 | 작업 유형 | `job_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | ITEM_GENERATION, DOCUMENT_GENERATION | `ITEM_GENERATION` |
| 1164 | 대상 엔터티 유형 | `target_entity_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_REVISION | `IQ_ITEM` |
| 1165 | 대상 엔터티 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 기존 대상이면 해당 개정 PK. 새 항목을 생성하는 미리보기는 NULL 허용 | `00000000-0000-0000-0000-000000000001` |
| 1166 | AI 모델명 | `model_name` | `varchar(100)` | N | N | - | N | - | N | Y | N | Y | 생성에 사용한 모델 식별자. 확인한 경우 기록하며 미확인 시 NULL을 허용한다. | - |
| 1167 | 입력 파라미터 | `input_parameters` | `jsonb` | N | N | - | Y | - | N | N | Y | Y | 화면 입력 목적·범위·기준 URS·생성 개수 또는 생성할 문서 목차 | `{"purpose":"접근권한 검증","scope":"로그인","count":3}` |
| 1168 | 생성 상태 | `generation_status` | `varchar(20)` | N | N | - | Y | `'PENDING'` | N | Y | N | Y | PENDING / PROCESSING / COMPLETED / FAILED / CANCELLED | `PENDING` |
| 1169 | 요청자 ID | `requested_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 생성 요청 사용자. 자동·배치 요청은 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1170 | 요청 시각 | `requested_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | Y | N | Y | 요청 접수 시각 | `2026-09-02T00:00:00Z` |
| 1171 | 실행 시작 시각 | `started_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 시작 시각 | `2026-09-02T00:00:00Z` |
| 1172 | 실행 완료 시각 | `completed_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | AI 처리 완료 시각 | `2026-09-02T00:00:00Z` |
| 1173 | 오류 코드 | `error_code` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 실패 원인 코드 | `LLM_TIMEOUT` |
| 1174 | 오류 메시지 | `error_message` | `text` | N | N | - | N | - | N | N | N | Y | 실패 상세 메시지 | `모델 응답시간 초과` |
| 1175 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1176 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 생성 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1177 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1178 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 수정 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `rule_ai_generation_job_1` | 업무 검증 | `target_entity_id` | ITEM_GENERATION의 신규 항목은 적용 전 target_entity_id NULL을 허용한다. DOCUMENT_GENERATION은 해당 문서 개정에 연결한다. |
| `rule_ai_generation_job_2` | 업무 검증 | - | 생성 완료가 업무 항목 승인이나 문서 승인을 의미하지 않는다. 선택·적용 후 일반 편집/승인 절차를 따른다. |
| `rule_ai_generation_job_3` | 업무 검증 | `project_id, requested_by` | 작업 생성 시 판정 기준 조직의 organization.ai_enabled가 TRUE여야 한다. 기준 조직은 project_id가 있으면 프로젝트의 시스템이 속한 조직, 없으면 요청자의 소속 조직이다. SYSTEM 계정은 SYSTEM 조직 소속(ai_enabled=FALSE)이므로 project_id 없는 AI 작업을 요청할 수 없다. FALSE이면 PENDING 행을 만들지 않고 생성을 거부한다. 감사대응 봇 질의도 같은 설정을 따르며 별도 저장 없이 서버에서 거부한다. ai_enabled를 FALSE로 전환할 때 이미 PENDING/PROCESSING인 작업은 완료까지 허용한다. |
| `ck_ai_generation_job_type` | CHECK | `job_type` | CHECK (job_type IN ('ITEM_GENERATION','DOCUMENT_GENERATION')) |
| `ck_ai_generation_job_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IN ('REQUIREMENT','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','DELIVERABLE_REVISION')) |
| `ck_ai_generation_job_status` | CHECK | `generation_status` | CHECK (generation_status IN ('PENDING','PROCESSING','COMPLETED','FAILED','CANCELLED')) |

[↑ 맨 위로](#top)

---

<a id="table-ai_generation_result"></a>
### 71. AI 생성 결과 (`ai_generation_result`)

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
| 1179 | AI 결과 ID | `ai_result_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | AI 결과 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1180 | AI 작업 ID | `ai_job_id` | `uuid` | N | Y | `ai_generation_job.ai_job_id` | Y | - | N | Y | N | Y | 상위 AI 작업 | `00000000-0000-0000-0000-000000000001` |
| 1181 | 결과 제목 | `result_title` | `varchar(300)` | N | N | - | N | - | N | N | N | Y | 결과 제목 | `OQ 테스트 초안` |
| 1182 | 선택 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 사용자 채택 여부 | `TRUE` |
| 1183 | 반영 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 실제 산출물 반영 여부 | `TRUE` |
| 1184 | 선택 사용자 ID | `selected_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | 채택 사용자 | `00000000-0000-0000-0000-000000000001` |
| 1185 | 선택 시각 | `selected_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 채택 시각 | `2026-09-02T00:00:00Z` |
| 1186 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1187 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 생성 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1188 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1189 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 수정 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `ck_ai_generation_result_selected` | CHECK | `is_selected, selected_by, selected_at` | CHECK (is_selected = FALSE OR (selected_by IS NOT NULL AND selected_at IS NOT NULL)) |

#### 업무 규칙

AI 결과 묶음이다. 항목별 선택·적용은 ai_result_item을 기준으로 하고 is_selected/is_applied는 해당 묶음의 요약이다.

[↑ 맨 위로](#top)

---

<a id="table-ai_result_item"></a>
### 72. AI 생성 결과 항목 (`ai_result_item`)

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
| 1190 | AI 결과 항목 ID | `ai_result_item_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 결과 항목 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1191 | AI 결과 ID | `ai_result_id` | `uuid` | N | Y | `ai_generation_result.ai_result_id` | Y | - | N | Y | N | Y | 상위 결과 참조 | `00000000-0000-0000-0000-000000000001` |
| 1192 | 항목 순번 | `item_order` | `integer` | N | N | - | Y | `1` | N | Y | N | Y | 결과 표시 순서 | `1` |
| 1193 | 항목 유형 | `item_type` | `varchar(50)` | N | N | - | Y | - | N | Y | N | Y | REQUIREMENT, FRA_SCENARIO, IQ_TEST, OQ_TEST, PQ_TEST, DOCUMENT_SECTION | `REQUIREMENT` |
| 1194 | 제목 | `title` | `varchar(500)` | N | N | - | N | - | N | N | N | Y | 생성 항목 제목 | `전자서명 기록` |
| 1195 | 본문 내용 | `content` | `text` | N | N | - | Y | - | N | N | N | Y | AI가 생성한 시험/요구사항/위험평가/목차별 초안 내용 | `내용` |
| 1196 | 적용 대상 유형 | `target_entity_type` | `varchar(50)` | N | N | - | N | - | N | Y | N | Y | 적용한 업무 테이블 유형: REQUIREMENT / FRA_ITEM / IQ_ITEM / OQ_ITEM / PQ_ITEM / DELIVERABLE_SECTION | `IQ_ITEM` |
| 1197 | 적용 대상 ID | `target_entity_id` | `uuid` | N | N | - | N | - | N | Y | N | Y | 실제로 적용하여 생성/수정한 개정 PK. 적용 전 NULL | `00000000-0000-0000-0000-000000000001` |
| 1198 | 채택 여부 | `is_selected` | `boolean` | N | N | - | Y | `FALSE` | N | Y | N | Y | 사용자 채택 여부 | `TRUE` |
| 1199 | 생성시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-02T00:00:00Z` |
| 1200 | 생성자 ID | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 생성 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1201 | 수정시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-02T00:00:00Z` |
| 1202 | 수정자 ID | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | 수정 사용자. 자동·배치 처리는 SYSTEM 계정의 user_id를 저장 | `00000000-0000-0000-0000-000000000001` |
| 1203 | 적용 여부 | `is_applied` | `boolean` | N | N | - | Y | `FALSE` | N | N | N | Y | 적용 여부 | `FALSE` |
| 1204 | 적용 시각 | `applied_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 적용 시각 | `2026-09-01T00:00:00Z` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_ai_result_item_1` | UNIQUE | `ai_result_id, item_order` | UNIQUE(ai_result_id,item_order) |
| `rule_ai_result_item_1` | 업무 검증 | `ai_result_id, item_order, is_applied` | is_applied=TRUE이면 실제 적용 대상 유형·PK와 applied_at이 필요하다 |
| `rule_ai_result_item_2` | 업무 검증 | - | 문서 목차별 결과는 DELIVERABLE_SECTION으로 연결한다. 상신(REVIEW) 이후의 항목·문서에는 적용하지 않으며 DRAFT 개정 또는 새 개정에만 적용한다. |
| `ck_ai_result_item_type` | CHECK | `item_type` | CHECK (item_type IN ('REQUIREMENT','FRA_SCENARIO','IQ_TEST','OQ_TEST','PQ_TEST','DOCUMENT_SECTION')) |
| `ck_ai_result_item_target_type` | CHECK | `target_entity_type` | CHECK (target_entity_type IS NULL OR target_entity_type IN ('REQUIREMENT','FRA_ITEM','IQ_ITEM','OQ_ITEM','PQ_ITEM','DELIVERABLE_SECTION')) |

[↑ 맨 위로](#top)

---

## Regulation

<a id="table-regulatory_source"></a>
### 73. 규정 근거 문서 (`regulatory_source`)

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
| 1205 | 규정 문서 ID | `regulatory_source_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1206 | 문서 코드 | `source_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 문서 코드 | `REF-001` |
| 1207 | 문서 유형 | `source_type` | `varchar(30)` | N | N | - | Y | - | N | N | N | Y | REGULATION / GUIDELINE / INTERNAL_SOP | `GUIDELINE` |
| 1208 | 문서명 | `title` | `varchar(255)` | N | N | - | Y | - | N | N | N | Y | 문서명 | `검토용 기준 문서` |
| 1209 | 판본/개정 | `edition` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 판본/개정 | `Rev.1` |
| 1210 | 발행기관/부서 | `issuing_body` | `varchar(200)` | N | N | - | N | - | N | N | N | Y | 발행기관/부서 | `품질보증팀` |
| 1211 | 출처 URL | `source_url` | `text` | N | N | - | N | - | N | N | N | Y | 출처 URL | `https://example.com/reference` |
| 1212 | 원본 첨부 ID | `file_id` | `uuid` | N | Y | `file_asset.file_id` | N | - | N | Y | N | Y | file_asset.file_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1213 | 적용 시작일 | `effective_from` | `date` | N | N | - | N | - | N | N | N | Y | 적용 시작일 | `2026-09-01` |
| 1214 | 적용 종료일 | `effective_to` | `date` | N | N | - | N | - | N | N | N | Y | 적용 종료일 | `2026-09-01` |
| 1215 | 검토 상태 | `status` | `varchar(20)` | N | N | - | Y | `'DRAFT'` | N | N | N | Y | DRAFT / VERIFIED / RETIRED | `DRAFT` |
| 1216 | 확인자 ID | `verified_by` | `uuid` | N | Y | `app_user.user_id` | N | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1217 | 확인 시각 | `verified_at` | `timestamptz` | N | N | - | N | - | N | N | N | Y | 확인 시각 | `2026-09-01T00:00:00Z` |
| 1218 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1219 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1220 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1221 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_regulatory_source_1` | UNIQUE | `source_code, edition` | UNIQUE(source_code,edition) |
| `rule_regulatory_source_1` | 업무 검증 | `source_code, edition` | 다른 언어판은 별도 source_code로 식별한다 |
| `rule_regulatory_source_2` | 업무 검증 | - | VERIFIED이면 확인자·시각 및 URL/원본 첨부 중 하나가 필요하다. 적용 종료일은 시작일 이전일 수 없다. |
| `rule_regulatory_source_3` | 업무 검증 | - | 새 판본은 새 행으로 등록한다. 승인된 업무가 참조한 문서판은 삭제하거나 새 내용으로 덮어쓰지 않는다. |
| `ck_regulatory_source_type` | CHECK | `source_type` | CHECK (source_type IN ('REGULATION','GUIDELINE','INTERNAL_SOP')) |
| `ck_regulatory_source_status` | CHECK | `status` | CHECK (status IN ('DRAFT','VERIFIED','RETIRED')) |

#### 업무 규칙

규정 근거는 재사용 기준 데이터로 관리한다. 수기 근거는 문서판과 조항을 확인한 후 연결하며, 검증 완료 상태는 확인자와 확인 시각을 기록한 경우에만 부여한다.

[↑ 맨 위로](#top)

---

<a id="table-regulatory_clause"></a>
### 74. 규정 조항 (`regulatory_clause`)

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
| 1222 | 조항 ID | `regulatory_clause_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1223 | 규정 문서 ID | `regulatory_source_id` | `uuid` | N | Y | `regulatory_source.regulatory_source_id` | Y | - | N | Y | N | Y | regulatory_source.regulatory_source_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1224 | 조항 코드 | `clause_code` | `varchar(100)` | N | N | - | Y | - | N | N | N | Y | 조항 코드 | `4.1` |
| 1225 | 조항 제목 | `title` | `varchar(255)` | N | N | - | N | - | N | N | N | Y | 조항 제목 | `접근 관리` |
| 1226 | 조항 요약 | `summary` | `text` | N | N | - | Y | - | N | N | N | Y | 조항 요약 | `해당 업무에 적용할 요구사항 요약` |
| 1227 | 원문 위치 | `source_locator` | `text` | N | N | - | N | - | N | N | N | Y | 원문 위치 | `제4장 1절` |
| 1228 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1229 | 사용 여부 | `is_active` | `boolean` | N | N | - | Y | `TRUE` | N | N | N | Y | 사용 여부 | `TRUE` |
| 1230 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1231 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1232 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1233 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_regulatory_clause_1` | UNIQUE | `regulatory_source_id, clause_code` | UNIQUE(regulatory_source_id,clause_code) |
| `ck_regulatory_clause_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_regulatory_clause_2` | 업무 검증 | - | 신규 선택은 VERIFIED 문서판의 활성 조항을 사용한다. 이미 승인에 사용한 조항이 비활성화되어도 과거 참조를 유지한다. |

[↑ 맨 위로](#top)

---

<a id="table-requirement_regulation"></a>
### 75. 요구사항 규정 근거 (`requirement_regulation`)

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
| 1234 | 근거 연결 ID | `requirement_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1235 | 업무 항목 ID | `requirement_id` | `uuid` | N | Y | `requirement.requirement_id` | Y | - | N | Y | N | Y | requirement.requirement_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1236 | 규정 조항 ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | regulatory_clause.regulatory_clause_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1237 | 적용 근거 | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | 적용 근거 | `접근권한 요구사항에 적용` |
| 1238 | 인용 문구 | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | 적용 당시 문서명·판본·조항 표시를 보존 | `검토용 기준 문서 Rev.1 / 4.1` |
| 1239 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1240 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1241 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1242 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1243 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_requirement_regulation_1` | UNIQUE | `requirement_id, regulatory_clause_id` | UNIQUE(requirement_id,regulatory_clause_id) |
| `ck_requirement_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_requirement_regulation_2` | 업무 검증 | - | 업무 승인 시 당시 조항과 인용 문구를 보존한다. 라이브러리에서 URS로 적용할 때 근거 관계도 복사하며 이후 템플릿 변경을 기존 URS에 자동 전파하지 않는다. |

[↑ 맨 위로](#top)

---

<a id="table-library_item_regulation"></a>
### 76. 라이브러리 규정 근거 (`library_item_regulation`)

| 항목 | 정의 |
|---|---|
| 설명 | 라이브러리 항목과 적용 규정 조항의 복수 연결 |
| Primary Key | `library_item_regulation_id` |
| 주요 참조(FK) | `library_id, regulatory_clause_id, created_by, updated_by` |
| GxP 중요도 | High |
| Audit 대상 | Y |
| 보존/삭제 원칙 | 감사 가능 기간 보존 |

#### 컬럼 정의

> `NN`: Not Null · `UQ`: Unique · `IDX`: Index · `민감`: 개인/민감정보

| No | 논리 컬럼명 | 물리 컬럼명 | 타입 | PK | FK | 참조 | NN | Default | UQ | IDX | 민감 | Audit | 설명/업무 규칙 | 예시값 |
|---:|---|---|---|:---:|:---:|---|:---:|---|:---:|:---:|:---:|:---:|---|---|
| 1244 | 근거 연결 ID | `library_item_regulation_id` | `uuid` | Y | N | - | Y | `gen_random_uuid()` | Y | Y | N | Y | 이 행의 고유 식별자 | `00000000-0000-0000-0000-000000000001` |
| 1245 | 라이브러리 항목 ID | `library_id` | `uuid` | N | Y | `library_item.library_id` | Y | - | N | Y | N | Y | library_item.library_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1246 | 규정 조항 ID | `regulatory_clause_id` | `uuid` | N | Y | `regulatory_clause.regulatory_clause_id` | Y | - | N | Y | N | Y | regulatory_clause.regulatory_clause_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1247 | 적용 근거 | `application_note` | `text` | N | N | - | N | - | N | N | N | Y | 적용 근거 | `접근권한 요구사항에 적용` |
| 1248 | 인용 문구 | `citation_snapshot` | `text` | N | N | - | Y | - | N | N | N | Y | 적용 당시 문서명·판본·조항 표시를 보존 | `검토용 기준 문서 Rev.1 / 4.1` |
| 1249 | 표시 순서 | `sort_order` | `integer` | N | N | - | Y | - | N | N | N | Y | 표시 순서 | `1` |
| 1250 | 생성 시각 | `created_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 생성 시각 | `2026-09-01T00:00:00Z` |
| 1251 | 생성자 | `created_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |
| 1252 | 수정 시각 | `updated_at` | `timestamptz` | N | N | - | Y | `CURRENT_TIMESTAMP` | N | N | N | Y | 수정 시각 | `2026-09-01T00:00:00Z` |
| 1253 | 수정자 | `updated_by` | `uuid` | N | Y | `app_user.user_id` | Y | - | N | Y | N | Y | app_user.user_id 참조 | `00000000-0000-0000-0000-000000000001` |

#### 제약조건

| 제약명 | 유형 | 적용 컬럼 | 조건 / 규칙 |
|---|---|---|---|
| `uq_library_item_regulation_1` | UNIQUE | `library_id, regulatory_clause_id` | UNIQUE(library_id,regulatory_clause_id) |
| `ck_library_item_regulation_sort_order` | CHECK | `sort_order` | CHECK (sort_order >= 1) |
| `rule_library_item_regulation_2` | 업무 검증 | - | 라이브러리 항목을 URS 등 업무 항목에 적용할 때 조항 연결과 인용 문구를 복사하며, 이후 라이브러리 근거 변경을 이미 적용한 업무 항목에 자동 전파하지 않는다. |

[↑ 맨 위로](#top)

---
