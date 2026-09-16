# JSON 업무 데이터 사전

이 문서는 [테이블 명세서](../TABLE_SPECIFICATION.md)의 상세 부록입니다. `demo_state.value` 안에 저장하는 업무 데이터를 TypeScript 선언과 실제 저장 코드 기준으로 설명합니다. 아래의 `SystemRecord`, `URSItem` 등은 **물리 데이터베이스 테이블이 아니라 JSON 객체 타입**입니다.

> **읽는 방법**: 먼저 저장 키 목록에서 관심 있는 업무를 찾은 다음 해당 타입의 필드 표를 확인하세요. 계정·프로젝트·요구사항 사이의 연결은 JSON 값으로 표현하는 업무 참조이며, 별도의 데이터베이스 외래 키를 뜻하지 않습니다.

## 바로 찾기

1. [표기와 저장 키](#notation)
2. [공통 승인·서명 구조](#common-approval)
3. [계정·권한·인벤토리·프로젝트](#admin-data)
4. [계획·평가·요구사항·설계](#planning-data)
5. [DQ·IQ/OQ/PQ·일탈](#qualification-data)
6. [RTM·VSR](#trace-data)
7. [산출물·승인본](#deliverable-data)
8. [프로젝트 종료·브라우저 캐시](#closure-cache)

<a id="notation"></a>

## 1. 표기와 저장 키

| 표기 | 의미 |
| --- | --- |
| `string`, `number`, `boolean` | JSON 문자열, 숫자, 참·거짓 값 |
| `Type[]` | 해당 타입의 값으로 이루어진 배열 |
| `Record<string, Type>` | 문자열 키에 해당 타입의 값을 대응시키는 객체 |
| 필드 이름 뒤의 `?` | TypeScript의 선택 필드. 저장된 JSON에 해당 필드가 없을 수 있으며, 데이터베이스의 `NULL` 허용 선언과는 다름 |
| `null` | 타입에 명시적으로 선언된 JSON `null` |
| 코드값 | 소스에 정의된 문자열 리터럴. 띄어쓰기 차이도 원문대로 표시 |
| 문서상 별칭 | 소스에서 이름 없이 중첩 선언한 객체를 설명하기 위해 이 문서에서 붙인 이름 |

`?`가 없는 필드는 TypeScript 선언상 필수입니다. 별도의 JSON 스키마 검증이나 DB 제약으로 강제된다는 뜻은 아닙니다. 이 문서는 현재 코드의 타입 계약을 설명하며 과거 저장 데이터는 일부 필드가 없거나 이전 형태일 수 있습니다. 날짜·시각은 대부분 `string`이며 ISO 형식과 화면 표시용 문자열이 함께 사용됩니다.

`{projectKey}`는 `projectId`를 우선 사용하고 없으면 프로젝트명을 사용한 정규화 값입니다. 영문을 소문자로 바꾸고 영문·숫자·한글 이외의 연속 문자를 `-`로 치환한 뒤 양 끝의 `-`를 제거합니다. 결과가 비면 `default`입니다. [키 생성 코드](../app/features/ValidationPlanningFeatures.tsx#L166)

### 1.1 전역 관리 데이터

| `demo_state.key` | `value` 형태 | 설명 |
| --- | --- | --- |
| `admin.system-inventory.v1` | [SystemRecord](#system-record)`[]` | 관리 대상 시스템·장비·시설 |
| `admin.system-history.v1` | [SystemHistory](#system-history)`[]` | 인벤토리 변경·승인 이력 |
| `admin.projects.v1` | [ManagedProject](#managed-project)`[]` | 프로젝트 및 기본 승인 워크플로 |
| `validation.approval-reset-events.v1` | [ApprovalResetEvent](#approval-reset-event)`[]` | 시스템 개정에 따른 프로젝트 승인 초기화 이력 |
| `admin.library.v1` | [LibraryStore](#library-store) | URS·FRA·IQ·OQ·PQ 재사용 라이브러리 |
| `admin.accounts.v1` | [AccountRecord](#account-record)`[]` | 업무 화면의 계정 정보 |
| `admin.permission-groups.v1` | [PermissionGroupRecord](#permission-group)`[]` | 권한 그룹과 구성원 |
| `admin.direct-permissions.v1` | [DirectPermissionGrant](#direct-permission)`[]` | 개인·그룹의 프로젝트 접근 권한 |
| `admin.system-permissions.v1` | [SystemPermissionGrant](#system-permission)`[]` | 계정별 관리 메뉴 권한 |
| `admin.permissions.inventory-approvers.v1` | [InventoryApprovalPolicy](#inventory-policy) | 인벤토리 작성·검토·승인·폐기 담당자 |

저장 호출: [인벤토리](../app/features/AdminProjectFeatures.tsx#L478), [라이브러리](../app/features/AdminProjectFeatures.tsx#L1825), [계정·권한](../app/features/AdminProjectFeatures.tsx#L3109), [프로젝트](../app/features/AdminProjectFeatures.tsx#L4916).

### 1.2 프로젝트 계획 데이터

| `demo_state.key` 패턴 | `value` 형태 | 설명 |
| --- | --- | --- |
| `validation.{projectKey}.vp` | [VPState](#vp-state) | 검증 계획서 |
| `validation.{projectKey}.va-assessment` | [VAAssessment](#va-assessment) | 현재 반영된 공급업체 평가 1건 |
| `validation.{projectKey}.va-assessments` | [VAAssessment](#va-assessment)`[]` | 공급업체 평가 목록 |
| `validation.{projectKey}.qia` | [QIAModule](#qia-module)`[]` | 품질 영향 평가 모듈·프로세스 |
| `validation.{projectKey}.qia-document` | [QIADocumentState](#qia-document) | QIA 문서의 버전·승인 |
| `validation.{projectKey}.urs` | [URSItem](#urs-item)`[]` | 사용자 요구사항 |
| `validation.{projectKey}.urs-document` | [URSDocumentState](#urs-document) | URS 문서의 버전·승인 대기 항목 |
| `validation.{projectKey}.fds` | [DesignState](#design-state) | FDS·DDS 설계 문서 목록 |
| `validation.{projectKey}.fra` | [FRAState](#fra-state) | 기능 위험 평가 |
| `approval-routes.{projectKey}.{scope}` | [ItemApprovalRouteStore](#item-approval-route) | VP·VA·QIA·URS·F&DS·FRA의 항목별 승인 경로 |

계획 데이터의 `{scope}`는 `vp`, `va`, `qia`, `urs`, `f&ds`, `fra`입니다. `f&ds`의 `&`는 이 키에서 그대로 유지됩니다. 같은 키 패턴의 `rtm`, `vsr`는 항목별 맵이 아닌 단일 승인 경로이므로 [승인 경로 구분](#item-approval-route)을 참고하세요.

저장 호출: [VP](../app/features/ValidationPlanningFeatures.tsx#L1486), [VA](../app/features/ValidationPlanningFeatures.tsx#L2020), [QIA](../app/features/ValidationPlanningFeatures.tsx#L2445), [URS](../app/features/ValidationPlanningFeatures.tsx#L2993), [F&DS](../app/features/ValidationPlanningFeatures.tsx#L3638), [FRA](../app/features/ValidationPlanningFeatures.tsx#L4039), [계획 승인 경로](../app/features/ValidationPlanningFeatures.tsx#L378).

<a id="common-approval"></a>

## 2. 공통 승인·서명 구조

<a id="approval-status"></a>

### 2.1 승인 상태와 단순 서명

계획 기능의 `ApprovalStatusValue`는 현재 상태 `작성중`, `검토중`, `승인중`, `승인완료`와 이전 상태 `초안`, `작성 중`, `검토 중`, `승인 완료`를 허용합니다. 화면에서는 상태를 정규화합니다. 인벤토리·시험 등 다른 타입은 자체 상태 코드가 있으므로 해당 필드 표를 따릅니다. [상태 선언·정규화](../app/features/ValidationPlanningFeatures.tsx#L175)

<a id="demo-signature"></a>
`DemoSignature`는 다음 두 필드로 구성됩니다. [소스](../app/features/ValidationPlanningFeatures.tsx#L60)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `signedBy` | `string` | 서명자 표시값 |
| `signedAt` | `string` | 서명 시각 |

<a id="approval-route-stage"></a>

### 2.2 ApprovalRouteStage — 승인 단계

소스: [ApprovalRouteSelector.tsx](../app/features/ApprovalRouteSelector.tsx#L26)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 단계 식별자 |
| `mode` | `string` | `직렬` / `병렬` |
| `signerIds` | `string[]` | 단계에 배정된 서명자 계정 ID |

승인 후보 표시용 `ApprovalRouteCandidate`는 `id`, `name`, `account`가 모두 `string`인 구조입니다. 후보 목록은 계정 정보를 바탕으로 구성하며 이 타입만을 위한 별도 저장 키는 없습니다. [소스](../app/features/ApprovalRouteSelector.tsx#L9)

<a id="unused-signature"></a>

### 2.3 UnusedSignature — 폐기 승인 서명

소스: [UnusedApprovalDialog.tsx](../app/features/UnusedApprovalDialog.tsx#L8)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `role` | `string` | `작성자` / `검토자` / `승인자` |
| `signedBy` | `string` | 서명자 표시값 |
| `signedAt` | `string` | 서명 시각 |
| `signerId?` | `string` | 계정 ID |
| `account?` | `string` | 계정 문자열 |
| `reason?` | `string` | 폐기 사유 |

<a id="item-approval-route"></a>

### 2.4 ItemApprovalRoute와 TraceApprovalRoute — 요청한 승인 경로

두 타입의 객체 필드는 동일하지만 **저장되는 최상위 형태가 다릅니다**.

| 업무 | 저장되는 `value` | 의미 |
| --- | --- | --- |
| VP·VA·QIA·URS·F&DS·FRA | `Record<string, ItemApprovalRoute>` | 항목 키별 승인 경로 맵. 코드상의 타입명은 `ItemApprovalRouteStore` |
| RTM·VSR | `TraceApprovalRoute` 또는 `null` | 해당 문서의 단일 승인 경로. 미요청 상태는 `null` |

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `stages?` | [ApprovalRouteStage](#approval-route-stage)`[]` | 단계별 승인 경로 |
| `mode?` | `string` | 이전 단일 단계 형태의 `직렬` / `병렬` |
| `signerIds?` | `string[]` | 이전 단일 단계 형태의 서명자 계정 ID |
| `signedIds` | `string[]` | 서명을 완료한 계정 ID |
| `requestedAt` | `string` | 승인 요청 시각 |

소스: [ItemApprovalRoute 선언·저장](../app/features/ValidationPlanningFeatures.tsx#L358), [TraceApprovalRoute 선언·저장](../app/features/TraceSummaryFeatures.tsx#L114).

<a id="admin-data"></a>

## 3. 관리 데이터 상세

<a id="system-record"></a>

### 3.1 SystemRecord — 시스템 인벤토리

저장 키: `admin.system-inventory.v1` · 형태: `SystemRecord[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L102)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 시스템 식별자 |
| `name` | `string` | 시스템명. 프로젝트의 `system`에서 이름으로 참조 |
| `type?` | `string` | 이전 데이터 호환용. 빈 문자열 / `생산 장비` / `품질 장비` / `유틸리티` / `IT 시스템` |
| `majorCategory` | `string` | 대분류 |
| `middleCategory` | `string` | 중분류 |
| `target` | `string` | 세부 관리 대상 |
| `managementNo` | `string` | 관리번호 |
| `department` | `string` | 관리 부서 |
| `location` | `string` | 설치·운영 위치 |
| `vendor` | `string` | 공급업체 |
| `model` | `string` | 모델명 |
| `description` | `string` | 설명 |
| `computerized` | `boolean` | 컴퓨터화 시스템 여부 |
| `softwareVersion` | `string` | 소프트웨어 버전 |
| `gamp` | `string` | 빈 문자열 / `Category 1` / `Category 2` / `Category 3` / `Category 4` |
| `gxp`, `part11` | `string` | 각각 GxP 및 Part 11 적용 구분. 빈 문자열 / `대상` / `비대상` |
| `linkedProjects` | `string[]` | 연결된 프로젝트의 **이름** 목록 |
| `version` | `number` | 인벤토리 개정 버전 |
| `approval` | `string` | `작성중` / `검토중` / `승인중` / `승인 완료` / `재승인 필요` / `승인 대기` |
| `updated` | `string` | 최근 갱신 일시 |
| `unused?` | `boolean` | 미사용·폐기 여부 |
| `unusedAt?`, `unusedReason?` | `string` | 각각 미사용 처리 시각·사유 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |
| `approvalRoute?` | `object` | 아래 인벤토리 승인 경로 |

`approvalRoute`의 중첩 필드:

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `reviewStages`, `approvalStages` | [ApprovalRouteStage](#approval-route-stage)`[]` | 각각 검토 단계·승인 단계 |
| `reviewerSignedIds`, `approverSignedIds` | `string[]` | 각각 검토·승인 서명을 완료한 계정 ID |

<details>
<summary>화면에서 사용하는 시스템 분류 코드</summary>

| 대분류 | 중분류 | 세부 대상 |
| --- | --- | --- |
| 컴퓨터화 시스템 | 품질 시스템 | LIMS, QMS, EDMS, 기타 |
| 컴퓨터화 시스템 | 생산 시스템 | MES, SCADA, DCS, 기타 |
| 컴퓨터화 시스템 | 전사 시스템 | ERP, 데이터 웨어하우스, 기타 |
| 장비 | 생산 장비 | 제조 설비, 포장 설비, 세척 설비, 기타 |
| 장비 | 품질 장비 | 분석 장비, 시험 장비, 환경 모니터링 장비, 기타 |
| 시설·유틸리티 | 시설 | 제조 구역, 보관 구역, 시험실, 기타 |
| 시설·유틸리티 | 유틸리티 | 정제수, 공조, 압축공기, 기타 |

필드의 TypeScript 타입은 `string`이며 위 값은 [화면 분류표](../app/features/AdminProjectFeatures.tsx#L137)입니다.

</details>

<a id="system-history"></a>

### 3.2 SystemHistory — 인벤토리 이력

저장 키: `admin.system-history.v1` · 형태: `SystemHistory[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L178)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 이력 식별자 |
| `systemId` | `string` | [SystemRecord.id](#system-record) 참조 |
| `at` | `string` | 이력 시각 |
| `version` | `number` | 해당 시스템 버전 |
| `mode` | `string` | `등록` / `저장` / `단순 저장` / `개정` / `승인` / `미사용` / `폐기` |
| `reason` | `string` | 처리 사유 |
| `approval` | `string` | [SystemRecord.approval](#system-record)과 동일한 코드값 |
| `approvedById?`, `approvedByName?` | `string` | 각각 승인자 계정 ID·이름 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 서명 |

<a id="managed-project"></a>

### 3.3 ManagedProject — 프로젝트

저장 키: `admin.projects.v1` · 형태: `ManagedProject[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L51)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 프로젝트 식별자. 프로젝트별 저장 키 생성의 우선 입력 |
| `name` | `string` | 프로젝트명 |
| `system` | `string` | 연결 시스템의 **이름**. [SystemRecord.name](#system-record) 참조 |
| `type` | `string` | `신규` / `변경` / `재검증` |
| `level` | `string` | `Level 1` / `Level 2` / `Level 3` / `사용자 지정` |
| `activities` | `string[]` | 선택된 검증 단계 코드 |
| `progress` | `number` | 진행률 |
| `status` | `string` | `진행중` / `정상종료` / `강제종료` |
| `comment` | `string` | 프로젝트 설명 |
| `updated` | `string` | 갱신 일시 |
| `workflow` | [PersistedApprovalWorkflow](#project-workflow) | 기본 승인 워크플로 |
| `stageWorkflows?` | `Record<string, PersistedApprovalWorkflow>` | 단계 코드별 승인 워크플로. 값은 [아래 구조](#project-workflow) |
| `history` | `ProjectHistory[]` | 아래 변경 이력 |

`ProjectHistory`의 모든 필드는 `string`입니다. [소스](../app/features/AdminProjectFeatures.tsx#L96)

| 필드 | 설명 |
| --- | --- |
| `at` | 발생 시각 |
| `action` | 수행 작업 |
| `detail` | 상세 내용 |

프로젝트와 시스템의 연결은 이름 기반입니다. 프로젝트 이름 변경 시 연결된 인벤토리의 `linkedProjects`도 갱신합니다. [연결 판정](../app/features/AdminProjectFeatures.tsx#L556), [이름 변경 반영](../app/features/AdminProjectFeatures.tsx#L5329)

<a id="project-workflow"></a>

### 3.4 프로젝트 승인 워크플로

`PersistedApprovalWorkflow`는 현재 `ApprovalWorkflow` 또는 이전 `LegacyApprovalWorkflow`입니다. [소스](../app/features/AdminProjectFeatures.tsx#L67)

| ApprovalWorkflow 필드 | 타입 | 설명 |
| --- | --- | --- |
| `authorStage` | `WorkflowStage` | 작성 단계 |
| `reviewStages`, `approvalStages` | `WorkflowStage[]` | 각각 검토 단계·승인 단계 |
| `applied` | `boolean` | 워크플로 적용 여부 |
| `signedBy?`, `signedAt?` | `string` | 각각 적용 서명자·서명 시각 |

| WorkflowStage 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 단계 식별자 |
| `mode` | `string` | `직렬` / `병렬` |
| `assignees` | `WorkflowAssignee[]` | 담당자·대체 담당자 목록 |

| WorkflowAssignee 필드 | 타입 | 설명 |
| --- | --- | --- |
| `userId` | `string` | 담당 계정 ID |
| `substituteUserId` | `string` | 대체 담당 계정 ID |

| LegacyApprovalWorkflow 필드 | 타입 | 설명 |
| --- | --- | --- |
| `mode` | `string` | `직렬` / `병렬` |
| `authors`, `reviewers`, `approvers` | `string[]` | 각각 작성자·검토자·승인자 계정 ID |
| `applied` | `boolean` | 적용 여부 |
| `signedBy?`, `signedAt?` | `string` | 각각 적용 서명자·서명 시각 |

<a id="approval-reset-event"></a>

### 3.5 ApprovalResetEvent — 시스템 개정에 따른 승인 초기화

저장 키: `validation.approval-reset-events.v1` · 형태: `ApprovalResetEvent[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L37)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 초기화 이벤트 ID |
| `occurredAt` | `string` | 발생 시각 |
| `trigger` | `string` | `SYSTEM_REVISION` |
| `scope` | `string` | `ALL_STAGE_APPROVALS` |
| `systemId`, `systemName` | `string` | 개정한 시스템의 ID·이름 |
| `systemVersion` | `number` | 개정한 시스템 버전 |
| `projectId`, `projectName` | `string` | 영향을 받은 프로젝트의 ID·이름 |
| `activityCodes` | `string[]` | 승인 초기화 대상 검증 단계 |
| `reason` | `string` | 초기화 사유 |

<a id="library-store"></a>

### 3.6 LibraryStore·LibraryItem — 재사용 라이브러리

저장 키: `admin.library.v1` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L1515)

`LibraryStore`는 `URS`, `FRA`, `IQ`, `OQ`, `PQ`를 키로 가지며 각 값은 `LibraryItem[]`입니다. 항목 타입이 공유되므로 해당 탭에서 쓰지 않는 문자열 필드는 빈 값일 수 있습니다.

| LibraryItem 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 라이브러리 내부 ID |
| `code` | `string` | 표시 코드 |
| `classification?` | `string` | `기본 제공` / `사용자 지정` |
| `majorCategory?`, `middleCategory?`, `target?` | `string` | 각각 대분류·중분류·대상 |
| `functionName?` | `string` | 기능명 |
| `scope?` | `string` | 적용 범위 |
| `category` | `string` | 분류 |
| `item` | `string` | 항목명 |
| `requirement` | `string` | 요구사항 |
| `acceptance` | `string` | 인수·판정 기준 |
| `regulation` | `string` | 관련 규정 |
| `testContent` | `string` | 시험 내용 |
| `expected` | `string` | 예상 결과 |
| `riskScenario?` | `string` | 위험 시나리오 |
| `sev?`, `occ?` | `number` | 각각 심각도·발생 가능성 |
| `det?` | `string` | 검출 가능성. `H` / `M` / `L` |
| `pi?`, `ll?`, `dl?` | `number` | 이전 위험 점수 호환 필드. 현재 FRA는 `sev`, `occ`, `det`를 우선 사용 |

<a id="account-record"></a>

### 3.7 AccountRecord — 업무 계정

저장 키: `admin.accounts.v1` · 형태: `AccountRecord[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L2570)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 업무 계정 ID |
| `account` | `string` | 로그인·표시용 계정 문자열 |
| `name` | `string` | 사용자명 |
| `groupId` | `string` | 이전 단일 그룹 ID |
| `groupIds?` | `string[]` | 복수 그룹 ID |
| `permissionLevel?` | `string` | `전체 관리자` / `프로젝트 관리자` / `사용자` |
| `workflowRole?` | `string` | 단일 역할. 아래 현재 역할 및 이전 `관리자`, `전체 관리자`, `조회자` 허용 |
| `workflowRoles?` | `string[]` | 현재 역할: `시스템 관리자`, `어드민`, `작성자`, `검토자`, `승인자`, `뷰어` |
| `menuPermissions?` | `Partial<Record<SystemPermissionMenu, MenuPermission>>` | [관리 메뉴별 권한](#menu-permission). 일부 메뉴가 없을 수 있음 |
| `status` | `string` | `활성` / `비활성` |
| `registered` | `string` | 등록 일자 |

계정이 속한 그룹을 계산할 때 권한 그룹의 `members`를 조회합니다. `groupId`는 이전 형태를 읽기 위한 보완 값입니다. [그룹 계산](../app/features/AdminProjectFeatures.tsx#L2641)

<a id="menu-permission"></a>

### 3.8 MenuPermission — 권한 값

소스: [메뉴와 권한 선언](../app/features/AdminProjectFeatures.tsx#L2589)

`SystemPermissionMenu`는 `시스템 인벤토리`, `프로젝트 관리`, `라이브러리`, `Audit Trail`, `계정 및 권한`입니다.

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `view` | `boolean` | 조회 허용 |
| `edit` | `boolean` | 편집 허용 |
| `dispose?` | `boolean` | 폐기 허용 |

`InventoryApprovalRoleKey`의 코드값은 `author`(작성), `reviewer`(검토), `approver`(승인), `disposer`(폐기)입니다. [선언](../app/features/AdminProjectFeatures.tsx#L219)

<a id="permission-group"></a>

### 3.9 PermissionGroupRecord — 권한 그룹

저장 키: `admin.permission-groups.v1` · 형태: `PermissionGroupRecord[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L2594)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 그룹 ID |
| `name` | `string` | 그룹명 |
| `description` | `string` | 설명 |
| `members` | `string[]` | [AccountRecord.id](#account-record) 목록 |
| `projects` | `string[]` | 프로젝트명 목록. 이전 `전체 프로젝트` 값 허용 |
| `menuPermissions` | `Record<SystemPermissionMenu, MenuPermission>` | [관리 메뉴별 권한](#menu-permission) |
| `projectPermissions?` | `Record<string, MenuPermission>` | [프로젝트별 권한](#menu-permission). 키는 프로젝트 ID 우선, 프로젝트명 호환 |
| `inventoryRoles?` | `InventoryApprovalRoleKey[]` | [인벤토리 역할 코드](#menu-permission) 목록 |

프로젝트 권한은 `projectPermissions[project.id]`를 먼저 확인하고 이름 키 및 이전 `projects` 배열을 보완적으로 사용합니다. [조회 코드](../app/features/AdminProjectFeatures.tsx#L3098)

<a id="direct-permission"></a>

### 3.10 DirectPermissionGrant — 프로젝트 권한 부여

저장 키: `admin.direct-permissions.v1` · 형태: `DirectPermissionGrant[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L2605)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 권한 부여 ID |
| `sourceType?` | `string` | 권한 출처. `direct` / `group` |
| `sourceId?` | `string` | 직접 부여 계정 또는 출처 그룹 ID |
| `originGroupId?` | `string` | 그룹에서 펼쳐진 권한의 원래 그룹 ID |
| `subjectType` | `string` | `개인` / `그룹` |
| `subjectId` | `string` | 대상 계정 또는 그룹 ID |
| `subjectName` | `string` | 대상 표시 이름 |
| `projectId?` | `string` | 대상 프로젝트 ID |
| `project` | `string` | 대상 프로젝트 표시값. 이전 데이터에서는 ID·이름 모두 비교 |
| `permissions?` | [MenuPermission](#menu-permission) | 조회·편집·폐기 권한 |

프로젝트 매칭은 `projectId`를 우선 사용합니다. 필드가 없는 이전 데이터는 `project`를 프로젝트 ID·이름과 비교합니다. [매칭 코드](../app/features/AdminProjectFeatures.tsx#L2682)

<a id="system-permission"></a>

### 3.11 SystemPermissionGrant — 관리 메뉴 권한 부여

저장 키: `admin.system-permissions.v1` · 형태: `SystemPermissionGrant[]` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L2619)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 권한 부여 ID |
| `accountId` | `string` | [AccountRecord.id](#account-record) 참조 |
| `accountName` | `string` | 계정 이름 |
| `sourceType` | `string` | `direct` / `group` |
| `sourceId` | `string` | `direct`이면 계정 ID, `group`이면 그룹 ID |
| `permissions` | `Record<SystemPermissionMenu, MenuPermission>` | [관리 메뉴별 권한](#menu-permission) |
| `inventoryRoles?` | `InventoryApprovalRoleKey[]` | [인벤토리 역할 코드](#menu-permission) 목록 |

생성 ID 형식은 `SYSGNT-{sourceType}-{sourceId}-{accountId}`입니다. [생성·호환 처리](../app/features/AdminProjectFeatures.tsx#L2966)

<a id="inventory-policy"></a>

### 3.12 InventoryApprovalPolicy — 인벤토리 승인 담당자 정책

저장 키: `admin.permissions.inventory-approvers.v1` · 소스: [타입 선언](../app/features/AdminProjectFeatures.tsx#L191)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `authors` | `PolicyMember[]` | 작성 담당자 |
| `reviewers` | `PolicyMember[]` | 검토 담당자 |
| `approvers` | `PolicyMember[]` | 승인 담당자 |
| `disposers?` | `PolicyMember[]` | 폐기 담당자 |
| `updatedAt` | `string` | 정책 갱신 시각 |

`PolicyMember`는 설명을 위한 **문서상 별칭**이며 실제 소스에는 같은 필드의 익명 객체가 반복 선언되어 있습니다.

| 중첩 필드 | 타입 | 설명 |
| --- | --- | --- |
| `accountId` | `string` | [AccountRecord.id](#account-record) 참조 |
| `name` | `string` | 이름 |
| `account` | `string` | 계정 문자열 |
| `status` | `string` | `활성` / `비활성` |

<a id="planning-data"></a>

## 4. 계획·평가·요구사항·설계 데이터 상세

<a id="vp-state"></a>

### 4.1 VPState — 검증 계획서

저장 키: `validation.{projectKey}.vp` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L1395)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `schemaVersion?` | `number` | 계획서 섹션 구조의 스키마 버전 |
| `version` | `string` | 문서 버전 |
| `status` | [ApprovalStatusValue](#approval-status) | 승인 상태 |
| `sections` | `VPSection[]` | 계획서 본문 섹션 |
| `approver` | `string` | 승인자 표시값 |
| `signature?` | [DemoSignature](#demo-signature) | 문서 승인 서명 |

| VPSection 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 섹션 ID |
| `title` | `string` | 제목 |
| `content` | `string` | 본문 |
| `enabled` | `boolean` | 섹션 포함 여부 |

<a id="va-assessment"></a>

### 4.2 VAAssessment — 공급업체 평가

저장 키: `validation.{projectKey}.va-assessment` 및 `validation.{projectKey}.va-assessments` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L1938)

목록 저장 후 현재 평가 1건도 별도 키에 반영합니다. 두 키는 독립된 물리 테이블이 아닙니다. [저장 코드](../app/features/ValidationPlanningFeatures.tsx#L2126)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 평가 내부 ID |
| `itemNo?` | `string` | 승인 시 부여되는 표시 번호 |
| `everApproved?` | `boolean` | 과거 승인 이력 존재 여부 |
| `unused?` | `boolean` | 폐기 여부 |
| `approvalVersion?` | `string` | 승인된 문서 버전 |
| `vendor` | `string` | 공급업체명 |
| `auditDate` | `string` | 감사 일자 |
| `auditor` | `string` | 감사자 표시값 |
| `attachment` | `string` | 첨부 파일명 |
| `approvalStatus?` | [ApprovalStatusValue](#approval-status) | 평가 항목 승인 상태 |
| `signature?` | [DemoSignature](#demo-signature) | 승인 서명 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |

업로드 성공 시 이 구조에는 `uploaded.name`만 저장됩니다. 파일의 DB ID나 저장소 키를 가진 직접 참조 필드는 없습니다. [첨부 반영](../app/features/ValidationPlanningFeatures.tsx#L2091)

<details>
<summary>이전 VA 데이터 호환 필드</summary>

`LegacyVAAssessment`는 위 필드에 아래 선택 필드를 추가합니다. 현재 정리 함수는 `summary`, `critical`, `major`, `minor`를 제거합니다.

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `status?`, `type?`, `typeDetail?` | `string` | 이전 상태·평가 유형 정보 |
| `summary?`, `critical?`, `major?`, `minor?` | `unknown` | 이전 findings 데이터. 현재 정리 대상 |

소스: [이전 타입·정리 함수](../app/features/ValidationPlanningFeatures.tsx#L1953), [정리 반영](../app/features/ValidationPlanningFeatures.tsx#L2041).

</details>

<a id="qia-module"></a>

### 4.3 QIAModule·QIAProcess — 품질 영향 평가

저장 키: `validation.{projectKey}.qia` · 형태: `QIAModule[]` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L2375)

| QIAModule 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 모듈 내부 ID |
| `itemNo?` | `string` | 승인 시 부여되는 표시 번호 |
| `everApproved?` | `boolean` | 과거 승인 이력 존재 여부 |
| `unused?` | `boolean` | 폐기 여부 |
| `approvalStatus?` | [ApprovalStatusValue](#approval-status) | 모듈 승인 상태 |
| `approvalVersion?` | `string` | 승인 버전 |
| `name` | `string` | 모듈명 |
| `description` | `string` | 설명 |
| `processes` | `QIAProcess[]` | 모듈 내부 프로세스 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |

| QIAProcess 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `id` | `string` | 프로세스 ID |
| `name` | `string` | 프로세스명 |
| `description` | `string` | 설명 |
| `answers` | `QAnswer[]` | 평가 문항 순서별 응답. `O` / `X` / `▲` |
| `part11?` | `PartAnswer[]` | Part 11 문항 순서별 응답. `Yes` / `No` |

<a id="qia-document"></a>

### 4.4 QIADocumentState — QIA 문서 상태

저장 키: `validation.{projectKey}.qia-document` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L2390)

| 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `version` | `string` | 문서 버전 |
| `status` | [ApprovalStatusValue](#approval-status) | 승인 상태 |
| `part11?` | `PartAnswer[]` | 문서 수준 Part 11 응답. `Yes` / `No` |
| `signature?` | [DemoSignature](#demo-signature) | 승인 서명 |

<a id="urs-item"></a>

### 4.5 URSItem — 사용자 요구사항

저장 키: `validation.{projectKey}.urs` · 형태: `URSItem[]` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L2851)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `uid` | `string` | 편집·선택에 사용하는 내부 항목 식별자 |
| `id` | `string` | 승인 시 부여되는 요구사항 번호. 미채번 초안은 빈 문자열 가능 |
| `version` | `string` | 요구사항 버전 |
| `category` | `string` | 요구사항 분류 |
| `name` | `string` | 요구사항명 |
| `regulation` | `string` | 관련 규정 |
| `requirement` | `string` | 요구사항 본문 |
| `criteria` | `string` | 판정 기준 |
| `source` | `string` | 등록 출처. 화면 모드에는 `직접 입력`, `라이브러리`, `시스템 패키지`, `AI 초안`이 있음 |
| `status` | [ApprovalStatusValue](#approval-status) | 승인 상태 |
| `approvedBy?`, `approvedAt?` | `string` | 각각 승인자·승인 시각 |
| `everApproved?` | `boolean` | 과거 승인 이력 존재 여부 |
| `unused?` | `boolean` | 폐기 여부 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |

후속 FRA 등의 `ursId`는 이 객체의 `id`를 참조하며 `uid`와는 용도가 다릅니다. 항목 폐기 시 번호·이력 관련 값은 남기고 요구사항 본문을 빈 문자열로 정리하므로, 필수 `string`이 항상 내용이 있는 문자열을 뜻하지는 않습니다. [초안 등록](../app/features/ValidationPlanningFeatures.tsx#L3062), [폐기 처리](../app/features/ValidationPlanningFeatures.tsx#L3190)

<a id="urs-document"></a>

### 4.6 URSDocumentState — URS 문서 버전

저장 키: `validation.{projectKey}.urs-document` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L2850)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `version` | `string` | 현재 문서 버전 |
| `pendingVersion?` | `string` | 다음 승인 대기 버전 |
| `pendingItemUids?` | `string[]` | 승인 대기 요구사항의 [URSItem.uid](#urs-item) 목록 |

<a id="design-state"></a>

### 4.7 DesignState·DesignDocument — FDS·DDS 설계 문서

저장 키: `validation.{projectKey}.fds` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L3579)

`DesignState`의 키는 `FDS`, `DDS`입니다. 각 값은 `DesignDocument` 1건 또는 `DesignDocument[]`이며, 화면은 이전 단일 객체도 배열로 정규화합니다. 신규 저장은 문서 배열을 사용합니다. [정규화](../app/features/ValidationPlanningFeatures.tsx#L3603), [파일 등록](../app/features/ValidationPlanningFeatures.tsx#L3734)

| DesignDocument 필드 | 타입 | 설명 |
| --- | --- | --- |
| `uid?` | `string` | 문서 내부 식별자 |
| `fileName` | `string` | 업로드 파일명 |
| `itemNo?` | `string` | 승인 시 부여되는 문서 번호 |
| `everApproved?` | `boolean` | 과거 승인 이력 존재 여부 |
| `unused?` | `boolean` | 폐기 여부 |
| `approvalVersion?` | `string` | 승인 버전 |
| `version` | `string` | 문서 버전 |
| `status` | [ApprovalStatusValue](#approval-status) | 승인 상태 |
| `uploadedAt` | `string` | 업로드 시각 |
| `owner` | `string` | 등록자 표시값 |
| `history` | `object[]` | 아래 파일 교체 이력 |
| `signature?` | [DemoSignature](#demo-signature) | 승인 서명 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |

| `history[]` 필드 | 타입 | 설명 |
| --- | --- | --- |
| `fileName` | `string` | 교체 전 파일명 |
| `version` | `string` | 교체 전 버전 |
| `replacedAt` | `string` | 교체 시각 |

현재 설계 문서 구조는 업로드 파일명만 보관하며 파일의 DB ID나 저장소 키를 직접 참조하지 않습니다. [업로드 결과 반영](../app/features/ValidationPlanningFeatures.tsx#L3743)

<a id="fra-state"></a>

### 4.8 FRAState·FRARow — 기능 위험 평가

저장 키: `validation.{projectKey}.fra` · 소스: [타입 선언](../app/features/ValidationPlanningFeatures.tsx#L3957)

| FRAState 필드 | 타입 | 설명 |
| --- | --- | --- |
| `status` | [ApprovalStatusValue](#approval-status) | 문서 승인 상태 |
| `version` | `string` | 문서 버전 |
| `rows` | `FRARow[]` | 위험 평가 항목 |
| `signature?` | [DemoSignature](#demo-signature) | 승인 서명 |
| `pendingVersion?` | `string` | 승인 대기 버전 |
| `pendingRowUids?` | `string[]` | 승인 대기 FRARow의 `uid` 목록 |
| `excludedDraftUrsIds?` | `string[]` | 초안 자동 구성에서 제외한 요구사항 번호 목록 |

| FRARow 필드 | 타입 | 설명·코드값 |
| --- | --- | --- |
| `uid` | `string` | 평가 항목 내부 ID |
| `ursId` | `string` | 연결된 [URSItem.id](#urs-item) |
| `ursVersion` | `string` | 연결한 요구사항의 버전 |
| `title` | `string` | 평가 제목 |
| `scenario` | `string` | 위험 시나리오 |
| `sev` | `number` | 심각도. 화면 계산 시 1~5로 제한 |
| `occ` | `number` | 발생 가능성. 화면 계산 시 1~5로 제한 |
| `det` | `string` | 검출 가능성. `H` / `M` / `L` |
| `pi?`, `ll?`, `dl?` | `number` | 이전 위험 점수. `sev`, `occ`, `det`가 없을 때 호환 계산에 사용 |
| `source` | `string` | `직접 입력` / `라이브러리` / `시스템 패키지` / `AI 초안` |
| `status` | [ApprovalStatusValue](#approval-status) | 항목 승인 상태 |
| `version?` | `string` | 항목 버전 |
| `everApproved?` | `boolean` | 과거 승인 이력 존재 여부 |
| `unused?` | `boolean` | 폐기 여부 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |

위험 우선순위 `RP`, 위험 등급 `RC`, 위험 우선순위 그룹 `RPG`, 조치 계획 등은 위 점수로 **계산되는 표시값**이며 이 `FRARow` 타입에 별도 필드로 선언되어 있지 않습니다. [점수·우선순위 계산](../app/features/ValidationPlanningFeatures.tsx#L3966)

<a id="qualification-data"></a>

## 5. 설계 적격성·시험·일탈 데이터

<a id="qualification-codes"></a>

### 5.1 공통 코드와 저장 키

| 코드 | 허용 값 |
| --- | --- |
| `Kind` | `IQ` / `OQ` / `PQ` |
| `Verdict` | 빈 문자열 / `Pass` / `Fail` / `N/A` |
| 시험 `Status` | `Draft` / `Approved` / `Executed` / `Deviation` / `Closed` |
| 시험·DQ·RTM·VSR `ApprovalStatus` | `작성중` / `검토중` / `승인중` / `승인완료` |
| 위 기능의 이전 승인 상태 | `초안` / `작성 중` / `검토 요청` / `검토 중` / `승인 중` / `승인 완료` |
| 시험 `Source` | `직접 등록` / `라이브러리` / `패키지` / `AI` |

| `demo_state.key` 패턴 | `value` 형태 |
| --- | --- |
| `{projectId}:dq-mappings` | [DQMapping](#dq-mapping)`[]` |
| `{projectId}:dq-mappings:approval-routes` | `Record<string, DQApprovalRoute>` |
| `{projectId}:iq-qualification` | [State](#qualification-state) |
| `{projectId}:oq-qualification` | [State](#qualification-state) |
| `{projectId}:pq-qualification` | [State](#qualification-state) |

이 영역의 `{projectId}`는 정규화하지 않은 프로젝트 ID이며, 미지정 시 `demo`를 사용합니다. 시험 상태와 승인 상태는 별도 필드입니다. 소스: [시험 타입](../app/features/QualificationTestFeatures.tsx#L58), [시험 저장](../app/features/QualificationTestFeatures.tsx#L681), [DQ 저장](../app/features/QualificationTraceFeatures.tsx#L597).

<a id="dq-mapping"></a>

### 5.2 DQMapping — 요구사항·설계 연결과 판정

소스: [타입 선언](../app/features/QualificationTraceFeatures.tsx#L281)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 매핑 내부 식별자 |
| `itemNo?` | `string` | 승인된 DQ 항목 번호 |
| `everApproved?`, `unused?` | `boolean` | 각각 과거 승인 여부·폐기 여부 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 서명 |
| `ursId`, `ursVersion`, `ursRequirement` | `string` | 요구사항 번호·버전·본문 스냅샷 |
| `fdsId?`, `fdsVersion?`, `fdsComment?` | `string` | FDS 식별값·버전·설명 |
| `ddsId?`, `ddsVersion?`, `ddsComment?` | `string` | DDS 식별값·버전·설명 |
| `verdict` | [Verdict](#qualification-codes) | 평가 결과 |
| `failReason` | `string` | 실패 사유 |
| `executor`, `executedAt` | `string` | 수행자·수행 시각 |
| `approvalStatus?` | 현재 또는 이전 승인 상태 | 항목 확인·승인 상태 |
| `approvalVersion?` | `string` | 승인 버전 |
| `designType?` | `string` | `FDS` / `DDS`, 이전 단일 연결 형태 |
| `designId?`, `designVersion?`, `designSection?`, `comment?` | `string` | 이전 설계 연결의 식별값·버전·절·설명 |

`DQApprovalRoute`는 [공통 승인 경로](#item-approval-route)의 필드와 같습니다. `stages?`, `mode?`, `signerIds?`, `signedIds`, `requestedAt`을 저장하며 항목별 맵으로 관리합니다. [선언](../app/features/QualificationTraceFeatures.tsx#L565), [저장](../app/features/QualificationTraceFeatures.tsx#L614)

<a id="qualification-state"></a>

### 5.3 State·Test — IQ/OQ/PQ 시험 상태

세 단계가 같은 데이터 구조를 사용하며 저장 키로 구분됩니다. 소스: [Test](../app/features/QualificationTestFeatures.tsx#L93), [State](../app/features/QualificationTestFeatures.tsx#L148)

| State 필드 | 타입 | 설명 |
| --- | --- | --- |
| `tests` | `Test[]` | 해당 단계의 시험 목록 |
| `deviations` | [Deviation](#deviation)`[]` | 해당 단계의 일탈 목록 |

| Test 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 시험 내부 ID. 일탈의 `testId`가 참조 |
| `itemNo?` | `string` | 승인 시 부여되는 시험 번호 |
| `everApproved?`, `unused?` | `boolean` | 과거 승인 여부·폐기 여부 |
| `disposedBy?`, `disposedAt?` | `string` | 폐기자·폐기 시각 |
| `disposalSignatures?` | [UnusedSignature](#unused-signature)`[]` | 폐기 승인 서명 |
| `approvalStatus?`, `finalApprovalStatus?` | 현재 또는 이전 승인 상태 | 각각 프로토콜 승인·결과 승인 상태 |
| `approvalVersion?` | `string` | 승인 버전 |
| `finalApprovalSignature?` | [시험 Signature](#test-signature) | 최종 결과 승인 서명 |
| `legacyApprovalMetadata?` | `{ itemNo?: string; everApproved?: boolean }` | 이전 승인 번호·승인 이력 호환 값 |
| `version`, `title` | `string` | 시험 버전·제목 |
| `ursIds` | `string[]` | 연결된 URS 업무 번호 목록 |
| `content`, `expected`, `acceptance` | `string` | 시험 내용·예상 결과·수용 기준 |
| `source` | [Source](#qualification-codes) | 시험 등록 출처 |
| `steps` | [Step](#test-attachment)`[]` | 수행 절차 |
| `status` | [Status](#qualification-codes) | 시험 진행 상태 |
| `approved` | `boolean` | 프로토콜 승인 여부 |
| `designSignature?` | [시험 Signature](#test-signature) | 프로토콜 서명 |
| `planApprovalActions?`, `finalApprovalActions?` | [ApprovalAction](#test-signature)`[]` | 프로토콜·결과 승인 처리 이력 |
| `actual` | `string` | 실제 수행 결과 |
| `attachments` | [Attachment](#test-attachment)`[]` | 이전 최상위 결과 첨부. 현재 증적은 `steps[].attachments` 사용 |
| `verdict` | [Verdict](#qualification-codes) | 시험 판정 |
| `performedOn` | `string` | 수행 날짜 |
| `executionSignature?` | [시험 Signature](#test-signature) | 수행 확인 서명 |
| `executionHistory?` | [ExecutionRecord](#execution-record)`[]` | 이전 수행 결과 보관 |
| `deviationReason`, `immediateAction` | `string` | 일탈 사유·즉시 조치 |
| `deviationId?` | `string` | 연결된 일탈 ID |
| `rerunCount` | `number` | 재수행 횟수 |

<a id="test-attachment"></a>

### 5.4 Attachment·Step — 첨부와 시험 절차

소스: [첨부·절차 선언](../app/features/QualificationTestFeatures.tsx#L80), [업로드 응답 타입](../app/features/uploadFile.ts#L3)

| Attachment 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id`, `key` | `string` | 업로드 파일 ID·R2 객체 키 |
| `name` | `string` | 원본 파일명 |
| `size` | `number` | 바이트 단위 크기 |
| `type` | `string` | MIME 타입. 빈 문자열 가능 |

| Step 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 절차 식별자 |
| `text` | `string` | 수행 절차 내용 |
| `confirmed` | `boolean` | 절차 확인 여부 |
| `attachments?` | `Attachment[]` | 절차별 첨부 |

시험 첨부는 업로드 ID·키를 보관하지만, VA와 FDS/DDS의 첨부 필드는 파일명만 보관합니다. 모든 업무 첨부가 동일한 참조 구조는 아닙니다.

[시험 복제 코드](../app/features/QualificationTestFeatures.tsx#L266)는 이전 최상위 `attachments`를 첫 절차 첨부의 대체값으로 사용하고 최상위 배열을 비웁니다. 현재 업로드는 [절차별 첨부 배열](../app/features/QualificationTestFeatures.tsx#L1709)에 저장합니다.

아래 두 필드는 `Attachment` 선언에는 없지만 [업로드 API 응답](../app/api/upload/route.ts#L39)에 포함됩니다. 클라이언트가 응답 객체 전체를 보관하므로 실제 첨부 JSON에 함께 남을 수 있습니다.

| 선언 외 필드 | 타입 | 설명 |
| --- | --- | --- |
| `storage` | `string` | `r2` / `memory`. 업로드 API의 저장 방식 표시 |
| `metadataOnly` | `boolean` | R2가 없으면 `true`. 이 경우 파일 본문은 보관하지 않음 |

<a id="test-signature"></a>

### 5.5 시험 Signature·ApprovalAction — 서명과 승인 처리

소스: [타입 선언](../app/features/QualificationTestFeatures.tsx#L66)

시험의 `Signature`는 `signedBy`, `signedAt`, `meaning`이 모두 `string`인 객체입니다. 계획 기능의 `DemoSignature`보다 서명 의미 `meaning`이 추가되어 있습니다.

`ApprovalAction`은 위 세 필드를 포함하며 아래 필드를 추가합니다.

| 추가 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id` | `string` | 처리 이력 ID |
| `scope` | `string` | `plan` / `final` / `deviation` |
| `from` | 현재 승인 상태 | 변경 전 상태 |
| `to` | `string` | `검토중` / `승인중` / `승인완료` |
| `actor` | `string` | `작성자` / `검토자` / `승인자` |
| `signerId?` | `string` | 서명자 계정 ID |
| `nextStages?`, `approvalStages?` | [ApprovalRouteStage](#approval-route-stage)`[]` | 후속 단계·승인 단계 지정 |
| `nextSignerIds?` | `string[]` | 이전 형식의 다음 서명자 목록 |
| `nextMode?` | `string` | `직렬` / `병렬` |

<a id="execution-record"></a>

### 5.6 ExecutionRecord — 이전 시험 수행 결과

소스: [타입 선언](../app/features/QualificationTestFeatures.tsx#L81)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `archivedAt` | `string` | 이력으로 보관한 시각 |
| `actual` | `string` | 이전 실제 결과 |
| `attachments` | [Attachment](#test-attachment)`[]` | 당시 첨부 |
| `verdict` | [Verdict](#qualification-codes) | 당시 판정 |
| `performedOn` | `string` | 당시 수행 날짜 |
| `executionSignature?` | [시험 Signature](#test-signature) | 당시 수행 서명 |
| `deviationReason`, `immediateAction` | `string` | 당시 일탈 사유·즉시 조치 |
| `deviationId?` | `string` | 당시 연결 일탈 ID |

<a id="deviation"></a>

### 5.7 Deviation — 일탈 처리

소스: [타입 선언](../app/features/QualificationTestFeatures.tsx#L130)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id`, `testId` | `string` | 일탈 ID·연결 시험 ID |
| `kind` | `string` | `IQ` / `OQ` / `PQ` |
| `reason`, `immediateAction` | `string` | 발생 사유·즉시 조치 |
| `status` | `string` | `사유·조치 승인 대기` / `재수행 가능` / `완료보고 대기` / `종료` |
| `createdBy`, `createdAt` | `string` | 등록자·등록 시각 |
| `approvalStatus?` | 현재 또는 이전 승인 상태 | 조치 승인 상태 |
| `approvalActions?` | [ApprovalAction](#test-signature)`[]` | 승인 처리 이력 |
| `actionSignature?` | [시험 Signature](#test-signature) | 사유·조치 승인 서명 |
| `rerunResult?` | [Verdict](#qualification-codes) | 재수행 판정 |
| `correctiveAction`, `completionReport` | `string` | 시정 조치·완료 보고 |
| `completionSignature?` | [시험 Signature](#test-signature) | 완료 서명 |
| `closureReason?` | `string` | 종료 사유 |

<a id="trace-data"></a>

## 6. 추적 매트릭스·종합 보고 데이터

<a id="rtm-row"></a>

### 6.1 RTMRow·LinkedTest — 요구사항 추적 매트릭스

저장 키: `{projectId}:rtm` · 형태: `RTMRow[]` · 소스: [타입 선언](../app/features/TraceSummaryFeatures.tsx#L238), [저장](../app/features/TraceSummaryFeatures.tsx#L452)

| RTMRow 필드 | 타입 | 설명 |
| --- | --- | --- |
| `id`, `itemNo` | `string` | 추적 행 ID·항목 번호 |
| `unused?` | `boolean` | 폐기 여부 |
| `ursId`, `ursVersion`, `ursText` | `string` | 요구사항 번호·버전·본문 |
| `latestUrs` | `boolean` | 최신 요구사항 여부 |
| `fraId`, `fraVersion` | `string` | 위험 평가 식별값·버전 |
| `fraFinalApproved` | `boolean` | 위험 평가 최종 승인 여부 |
| `actionType` | `string` | `No Action` / `SOP` / `Test` / `미분류` |
| `sopId` | `string` | 연결 SOP 식별값 |
| `designType?` | `string` | `FDS` / `DDS` |
| `designId`, `designVersion` | `string` | 설계 식별값·버전 |
| `designLatestApproved` | `boolean` | 설계의 최신 승인 여부 |
| `dqId` | `string` | DQ 식별값 |
| `dqItemNo?` | `string` | DQ 항목 번호 |
| `dqVerdict` | [Verdict](#qualification-codes) | DQ 판정 |
| `tests` | `LinkedTest[]` | 연결 시험 요약 |

| LinkedTest 필드 | 타입 | 설명 |
| --- | --- | --- |
| `kind` | `string` | `IQ` / `OQ` / `PQ` |
| `id`, `version` | `string` | 시험 ID·버전 |
| `itemNo?` | `string` | 시험 항목 번호 |
| `verdict` | `string` | `Pass` / `Fail` / `N/A` |
| `finalApproved` | `boolean` | 결과 최종 승인 여부 |

<a id="rtm-document"></a>

### 6.2 RTMDocumentState — RTM 문서 상태

저장 키: `{projectId}:rtm-document` · 소스: [타입 선언](../app/features/TraceSummaryFeatures.tsx#L427)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `status` | 현재 또는 이전 승인 상태 | 문서 확인·승인 상태 |
| `version` | `string` | 문서 버전 |
| `sourceSignature` | `string` | 원본 데이터 변경 비교용 문자열. 사람의 전자서명이 아님 |
| `signature?` | [DemoSignature와 같은 구조](#demo-signature) | 확인자·서명 시각 |

<a id="vsr-state"></a>

### 6.3 VSRState·Activity — 검증 요약 보고

저장 키: `{projectId}:vsr` · 소스: [VSRState](../app/features/TraceSummaryFeatures.tsx#L758), [Activity](../app/features/TraceSummaryFeatures.tsx#L676)

| VSRState 필드 | 타입 | 설명 |
| --- | --- | --- |
| `activities` | `Activity[]` | 단계별 종합 현황 |
| `lastGeneratedAt` | `string` | 마지막 생성 시각 |
| `version` | `string` | 보고서 버전 |
| `status` | 현재 또는 이전 승인 상태 | 승인 상태 |
| `sourceSignature` | `string` | 원본 변경 비교용 문자열 |
| `signature?` | [DemoSignature와 같은 구조](#demo-signature) | 확인자·서명 시각 |

| Activity 필드 | 타입 | 설명 |
| --- | --- | --- |
| `key`, `activity` | `string` | 단계 키·활동명 |
| `selected` | `boolean` | 보고서 포함 여부 |
| `documentNo`, `version` | `string` | 문서 번호·버전 |
| `deviations` | `string` | 일탈 요약 |
| `approver`, `approvalDate` | `string` | 승인자·승인 날짜 |
| `finalApproved` | `boolean` | 최종 승인 여부 |
| `sourceFingerprint?` | `string` | 원본 변경 감지용 식별 문자열 |

RTM·VSR의 승인 경로는 `approval-routes.{projectKey}.rtm`, `approval-routes.{projectKey}.vsr`에 별도 저장하며 [TraceApprovalRoute](#item-approval-route) 또는 `null`입니다. 추적 화면의 `SourceURS`, `SourceFRA` 등 `Source*` 타입은 기존 저장 데이터를 읽기 위한 부분 타입이므로 별도 데이터셋으로 중복 명세하지 않습니다.

<a id="deliverable-data"></a>

## 7. 산출물·승인본 데이터

<a id="workspace-store"></a>

### 7.1 WorkspaceStore — 단계별 문서 작업 공간

저장 키: `deliverable-document:{projectId}:{stage}:v1` · 소스: [타입 선언](../app/features/DeliverableDocumentWorkspace.tsx#L216), [저장](../app/features/DeliverableDocumentWorkspace.tsx#L1538)

`{stage}`는 `vp`, `va`, `qia`, `urs`, `fands`, `fra`, `dq`, `iq`, `oq`, `pq`, `rtm`, `vsr`입니다. F&DS는 여기에서 `fands`로 변환됩니다. 계획 승인 경로의 `f&ds`와 혼동하지 않아야 합니다.

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `draft` | [GeneratedDraft](#generated-draft) 또는 `null` | 현재 문서 초안 |
| `documentVersions?` | [GeneratedDraft](#generated-draft)`[]` | 저장된 문서 버전 이력 |
| `approvalReleases?` | [ApprovalReleases](#approval-releases) | 승인 기준 데이터의 버전별 스냅샷 |

<a id="generated-draft"></a>

### 7.2 GeneratedDraft — 문서 초안·개정본

소스: [타입 선언](../app/features/DeliverableDocumentWorkspace.tsx#L198)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `title`, `version`, `updatedAt` | `string` | 제목·문서 버전·변경 시각 |
| `approvedVersion?` | `string` | 근거가 된 승인 데이터 버전 |
| `approvedTableSnapshot?` | [ApprovedDataTable](#document-section) | 승인 데이터 표 |
| `part11TableSnapshot?` | [ApprovedDataTable](#document-section) | Part 11 관련 표 |
| `revisionTableSnapshot?` | [ApprovedDataTable](#document-section) | 개정 이력 표 |
| `structureVersion?` | `number` | 허용 값 `2` / `3`, 문서 구성 형식 버전 |
| `sections` | [DocumentSection](#document-section)`[]` | 문서 섹션. 실제 저장 객체에는 `source`도 포함될 수 있음 |
| `sectionOrder?` | `string[]` | 섹션 표시 순서 |
| `approvalStatus?` | 현재 승인 상태 | `작성중` / `검토중` / `승인중` / `승인완료` |
| `approvalActions?` | [DeliverableApprovalAction](#deliverable-approval-action)`[]` | 산출물 승인 이력 |
| `reapprovalSourceStage?` | `DeliverableStage` | 재승인을 유발한 상위 단계. 대문자 코드, F&DS 포함 |
| `reapprovalReason?`, `reapprovalInvalidatedAt?` | `string` | 재승인 사유·승인 무효화 시각 |

<a id="document-section"></a>

### 7.3 DocumentSection·ApprovedDataTable — 문서 구성

소스: [섹션](../app/features/DeliverableDocumentWorkspace.tsx#L63), [표](../app/features/DeliverableDocumentWorkspace.tsx#L134)

| 타입 | 필드 | 타입·설명 |
| --- | --- | --- |
| `DocumentSection` | `id`, `title`, `content` | 모두 `string`. 섹션 ID·제목·본문 |
| `ApprovedDataTable` | `columns` | `string[]`. 열 제목 |
| `ApprovedDataTable` | `rows` | `string[][]`. 행별 문자열 셀 |
| `StoredDocumentSection` | 위 섹션 필드 + `source?` | `source`는 `ai` / `manual` |
| `AuthorSection` | 위 섹션 필드 + `source` | `history` / `approved` / `part11` / `ai` / `manual` |

`GeneratedDraft.sections`의 선언 타입은 `DocumentSection[]`이지만, [실제 초안 저장 코드](../app/features/DeliverableDocumentWorkspace.tsx#L2242)는 `source`가 `ai` 또는 `manual`인 객체도 그대로 저장합니다. 따라서 저장 JSON을 읽을 때 `StoredDocumentSection`의 `source` 필드가 존재할 수 있습니다.

`AuthorSection`은 편집·구성에 쓰는 타입이며 전용 DB 키가 없습니다. 브라우저 작업본은 편집 가능한 `ai`, `manual` 섹션만 보관합니다.

<a id="approval-releases"></a>

### 7.4 ApprovalReleases·ApprovalRelease — 승인 데이터 스냅샷

소스: [타입 선언](../app/features/DeliverableDocumentWorkspace.tsx#L139)

| ApprovalReleases 필드 | 타입 | 설명 |
| --- | --- | --- |
| `schemaVersion` | `number` | 고정 값 `1` |
| `currentVersion` | `string` | 현재 승인 데이터 버전 |
| `versions` | `ApprovalRelease[]` | 버전별 승인 데이터 |
| `invalidatedAt?` | `string` | 승인 무효화 시각 |
| `invalidatedByStage?` | `DeliverableStage` | 무효화를 유발한 단계 |
| `invalidatedReason?` | `string` | 무효화 사유 |

| ApprovalRelease 필드 | 타입 | 설명 |
| --- | --- | --- |
| `version` | `string` | 승인 데이터 버전 |
| `approvedAt`, `approvedBy` | `string` | 승인 시각·승인자 |
| `columns` | `string[]` | 승인 데이터 열 제목 |
| `rows` | `string[][]` | 승인된 항목의 값 |

업무 항목의 승인 데이터인 `approvalReleases`와 생성한 산출물 자체의 `draft.approvalStatus`는 별도 상태입니다.

<a id="deliverable-approval-action"></a>

### 7.5 DeliverableApprovalAction — 산출물 승인 이력

소스: [타입 선언](../app/features/DeliverableDocumentWorkspace.tsx#L175)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `role` | `string` | `작성자` / `검토자` / `승인자` |
| `signerId?` | `string` | 서명자 계정 ID |
| `signedBy`, `signedAt` | `string` | 서명자 표시값·서명 시각 |
| `nextStages?`, `approvalStages?` | [ApprovalRouteStage](#approval-route-stage)`[]` | 다음 역할의 단계·승인 단계 |
| `nextSignerIds?` | `string[]` | 이전 형식의 다음 서명자 ID |
| `nextMode?` | `string` | `직렬` / `병렬` |

<a id="closure-cache"></a>

## 8. 프로젝트 종료·브라우저 보관 데이터

<a id="project-closure"></a>

### 8.1 ProjectClosureRequest — 프로젝트 종료 요청

저장 키: `project-closure:{projectId}:{kind}:v1` · 형태: `ProjectClosureRequest` 또는 `null` · 소스: [타입 선언](../app/features/ProjectClosureWorkflow.tsx#L14), [저장](../app/features/ProjectClosureWorkflow.tsx#L92)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| `projectId`, `projectName` | `string` | 대상 프로젝트 ID·이름 |
| `kind` | `string` | `정상종료` / `강제종료`. 저장 키의 `{kind}`와 대응 |
| `status` | `string` | `작성중` / `검토중` / `승인중` / `완료` |
| `progress` | `number` | 요청 당시 진행률 |
| `reason` | `string` | 종료 사유 |
| `requestedAt` | `string` | 요청 시각 |
| `signatures` | `ProjectClosureSignature[]` | 종료 승인 서명 이력 |

| ProjectClosureSignature 필드 | 타입 | 설명 |
| --- | --- | --- |
| `role` | `string` | `작성자` / `검토자` / `승인자` |
| `signerId`, `signedBy`, `signedAt` | `string` | 계정 ID·서명자 표시값·서명 시각 |
| `reviewStages?`, `approvalStages?` | [ApprovalRouteStage](#approval-route-stage)`[]` | 검토 경로·승인 경로 |

<a id="browser-cache"></a>

### 8.2 브라우저 localStorage — DB 테이블이 아닌 복구용 데이터

| 브라우저 키 | 내용 | 근거 |
| --- | --- | --- |
| `validocs:persistent-state:{epoch}:{demoStateKey}` | `{ value, updatedAt, pending }` | [영속 상태 훅](../app/features/usePersistentDemoState.ts#L20) |
| `validocs:demo-data-epoch` | 현재 데모 데이터 세대 문자열 | [세대 마커](../app/features/usePersistentDemoState.ts#L23) |
| `validocs:deliverable-working:{projectId}:{stage}:v1` | `WorkingDocumentCache` | [산출물 작업본](../app/features/DeliverableDocumentWorkspace.tsx#L1539) |

`epoch`는 [DEMO_DATA_EPOCH](../app/demoDataEpoch.ts)에 정의됩니다. 첫 번째 구조의 `value`는 해당 업무 JSON, `updatedAt`은 `string`, `pending`은 서버 저장 완료 전 여부인 `boolean`입니다.

| WorkingDocumentCache 필드 | 타입 | 설명 |
| --- | --- | --- |
| `sourceVersion` | `string` | 작업본의 근거 승인 버전 |
| `sections` | 섹션 객체 배열 | `id`, `title`, `content`는 `string`, `source`는 `ai` / `manual`로 필수 |
| `updatedAt` | `string` | 작업본 저장 시각 |

소스: [작업본 타입 및 읽기·쓰기](../app/features/DeliverableDocumentWorkspace.tsx#L79). 브라우저 캐시는 D1 테이블이 아니며, 서버 메모리 대체 저장소와도 별개입니다.

[테이블 명세서로 돌아가기](../TABLE_SPECIFICATION.md)

