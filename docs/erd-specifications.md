## 1. 조직·사용자·전역 권한 관리
<img width="1650" height="1066" alt="image" src="https://github.com/user-attachments/assets/d1f9d215-dafa-46ae-b7ff-75f4bad5b622" />
Link : https://drawsql.app/teams/minho-kim/diagrams/01-organization-user-and-global-role-management

### 구조 개요

조직을 기준으로 사용자 계정을 관리하고, 역할 마스터와 사용자 역할 매핑을 통해 사용자별 전역 권한을 부여하는 구조입니다.

전역 역할은 시스템 전체에 적용되는 권한이며, 프로젝트별 참여 여부와 수행 역할은 별도의 `project_member` 테이블에서 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 고객사 또는 운영 조직의 기본정보를 관리하고 사용자 소속을 구분하는 기준 테이블 |
| `app_user` | 사용자 계정, 기본 프로필, 소속 조직 및 계정 상태 관리 |
| `role` | 시스템 전체에 적용되는 전역 역할과 권한 범위 정의 |
| `user_role` | 사용자와 전역 역할 간의 다대다 매핑 및 동일 역할 중복 부여 방지 |

---
## 2. 시스템·프로젝트·참여자 관리
<img width="2013" height="1579" alt="image" src="https://github.com/user-attachments/assets/ad1f10d8-fd53-4e0d-9363-ead6d592dd9a" />
Link : https://drawsql.app/teams/minho-kim/diagrams/02-system-project-and-participant-management

### 구조 개요

조직에 소속된 시스템과 장비를 기준으로 Validation 프로젝트를 구성하고, 프로젝트별 참여 사용자와 수행 역할을 관리하는 구조입니다.

각 프로젝트는 하나의 대상 시스템에 연결되며, `project_member`를 통해 사용자와 역할을 프로젝트 단위로 매핑합니다. 이를 통해 동일한 사용자도 프로젝트마다 서로 다른 역할을 수행할 수 있습니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 관리번호, 유형, 담당부서, GAMP 범주, GxP 구분 및 식별 상태 관리 |
| `validation_project` | 대상 시스템별 Validation 프로젝트의 범위, 검증 방식, 진행률, 상태 및 GAMP 카테고리 관리 |
| `project_member` | 프로젝트별 참여 사용자와 수행 역할을 연결하고 참여 상태 및 참여 기간 관리 |
| `app_user` | 프로젝트에 참여하거나 프로젝트를 생성·수정하는 사용자 계정과 기본 프로필 관리 |
| `role` | 프로젝트 참여자에게 부여할 작성자, 검토자, 승인자 등의 역할 기준정보 관리 |


---
## 3. Validation 활동·선후행 조건 관리
<img width="2500" height="1729" alt="image" src="https://github.com/user-attachments/assets/3648a194-0317-43eb-aa29-fd262b715add" />
Link : https://drawsql.app/teams/minho-kim/diagrams/03-validation-activity-and-dependency-management

### 구조 개요

조직에 소속된 시스템을 기준으로 Validation 프로젝트를 생성하고, 프로젝트별 수행 활동과 활동 간 선후행 조건을 관리하는 구조입니다.

`validation_activity`에서 전체 Validation 활동 기준을 정의하고, `project_activity`에서 프로젝트별 수행 대상, 필수 여부 및 진행 상태를 관리합니다. `activity_dependency`는 선행 활동의 상태나 업무 조건에 따라 후행 활동의 활성화 여부를 판정하는 기준을 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트와 활동 및 선후행 조건을 등록·수정하는 사용자 정보 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 식별정보, GAMP 범주, GxP 구분 및 상태 관리 |
| `validation_project` | 대상 시스템별 Validation 프로젝트의 범위, 검증 방식, 진행률 및 상태 관리 |
| `validation_activity` | VP, QIA, VA, URS, FDS, DDS, DQ, FRA, IQ, OQ, PQ, RTM, VSR 등 Validation 활동의 기준정보와 기본 표시 순서 관리 |
| `project_activity` | 프로젝트별 수행 대상 활동, 필수 여부, 활성화 여부 및 진행 상태 관리 |
| `activity_dependency` | 선행 활동, 요구 상태, 조건 유형 및 평가 순서를 기준으로 후행 활동의 활성화 조건 관리 |
