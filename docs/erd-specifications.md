## 조직·사용자·전역 권한 관리
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
