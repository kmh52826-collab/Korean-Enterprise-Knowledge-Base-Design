# ERD Specifications

Validation Management Platform의 전체 데이터 모델을 업무 영역별로 구분하여 정리한 ERD 명세서입니다.

복잡한 전체 스키마를 16개 주제로 분리하고, 각 영역의 주요 테이블과 관계를 ERD 이미지로 제공합니다. 각 섹션에는 해당 영역의 구조 개요와 테이블별 역할을 함께 정리했습니다.

> [!NOTE]
> 이 저장소는 데이터 모델링 및 시스템 설계 사례를 공유하기 위한 공개용 자료입니다.
>
> 문서에 사용된 조직명, 사용자 정보, 시스템명, 프로젝트명, 테이블명, 컬럼명 및 예시값 등은 특정 회사의 실제 운영 데이터나 내부 시스템 구조를 나타내지 않습니다. 공개 목적에 맞게 일반화하거나 임의로 구성한 명칭과 예시를 사용했습니다.
>
> 실제 운영 환경에 적용할 때는 조직의 업무 정책, 보안 기준, 개인정보 보호 요건, 데이터 보존 정책 및 관련 규제 요구사항을 별도로 검토해야 합니다.

> **상세 컬럼 정보가 필요한 경우**  
> 각 테이블의 컬럼, 데이터 타입, PK·FK, Null 허용 여부, 기본값 및 업무 규칙은 [Data Dictionary](./data-dictionary.md)를 참고하세요.

## 목차

1. [조직·사용자·전역 권한 관리](#1-조직사용자전역-권한-관리)
2. [시스템·프로젝트·참여자 관리](#2-시스템프로젝트참여자-관리)
3. [Validation 활동·선후행 조건 관리](#3-validation-활동선후행-조건-관리)
4. [라이브러리·QIA·공급업체 감사 관리](#4-라이브러리qia공급업체-감사-관리)
5. [URS·FDS 관리](#5-ursfds-관리)
6. [DDS·DQ 관리](#6-ddsdq-관리)
7. [FRA 위험평가 관리](#7-fra-위험평가-관리)
8. [IQ·OQ·PQ 적격성 시험 관리](#8-iqoqpq-적격성-시험-관리)
9. [설계·위험 산출물 추적성 관리](#9-설계위험-산출물-추적성-관리)
10. [적격성 시험 추적성·RTM 관리](#10-적격성-시험-추적성rtm-관리)
11. [VSR·일탈 관리](#11-vsr일탈-관리)
12. [Workflow·승인·전자서명 관리](#12-workflow승인전자서명-관리)
13. [파일·증적·파일 정리 관리](#13-파일증적파일-정리-관리)
14. [리포트·알림·백업 운영 관리](#14-리포트알림백업-운영-관리)
15. [AI 생성 작업·결과 관리](#15-ai-생성-작업결과-관리)
16. [Audit Trail 관리](#16-audit-trail-관리)

---

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
<img width="2000" height="1491" alt="image" src="https://github.com/user-attachments/assets/b8625331-f875-41a2-9546-dc2ef32557cd" />
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

---

## 4. 라이브러리·QIA·공급업체 감사 관리
<img width="2600" height="1866" alt="image" src="https://github.com/user-attachments/assets/0fa424a7-ec2f-4589-8b5e-75b20e7eefa0" />
Link : https://drawsql.app/teams/minho-kim/diagrams/04-library-qia-and-vendor-audit-management

### 구조 개요

조직에 소속된 시스템을 기준으로 Validation 프로젝트를 구성하고, 재사용 가능한 표준 라이브러리와 프로젝트별 품질 영향 평가 및 공급업체 감사 결과를 관리하는 구조입니다.

`library_item`은 URS·IQ·OQ 작성에 활용할 표준 항목을 관리합니다. `qia_assessment`와 `qia_module_item`은 프로젝트의 GxP 및 21 CFR Part 11 적용 범위를 평가하며, `vendor_audit`은 대상 시스템 공급업체의 감사 계획과 결과 및 결함 수를 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트와 평가 문서를 작성·관리하는 사용자 계정 및 기본 프로필 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 식별정보, 공급업체, GAMP 범주, GxP 구분 및 상태 관리 |
| `validation_project` | 대상 시스템별 Validation 프로젝트의 범위, 검증 방식, 진행률 및 상태 관리 |
| `library_item` | URS·IQ·OQ에서 재사용할 표준 요구사항, 시험 절차, 기대 결과, 수용 기준 및 근거 규정 관리 |
| `qia_assessment` | 프로젝트별 21 CFR Part 11 적용 여부와 GxP 범위, 문서 버전 및 평가 상태 관리 |
| `qia_module_item` | QIA 문서의 모듈·프로세스별 GxP 세부 평가 항목과 평가 결과 관리 |
| `vendor_audit` | 공급업체 감사 방식, 일정, 감사 결과, 결함 수, 문서 버전 및 진행 상태 관리 |

---

## 5. URS·FDS 관리
<img width="4440" height="3366" alt="image" src="https://github.com/user-attachments/assets/a424ad13-1d60-4e0d-8f99-97d72d1926c1" />
Link : https://drawsql.app/teams/minho-kim/diagrams/05-urs-and-fds-management

### 구조 개요

조직에 소속된 시스템을 기준으로 Validation 프로젝트를 구성하고, 프로젝트별 사용자 요구사항과 기능 설계 명세를 작성·관리하는 구조입니다.

`requirement`에서 URS 요구사항과 개정 버전을 관리하고, `fds_spec`에서 FDS 문서의 버전과 상태를 관리합니다. `fds_item`과 `fds_interface`에서는 FDS 문서에 포함되는 기능·화면 항목과 시스템 간 인터페이스 설계를 각각 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트, URS 및 FDS 문서와 상세 항목을 작성·수정하는 사용자 정보 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 식별정보, 관리번호, GAMP 범주, GxP 구분 및 상태 관리 |
| `validation_project` | URS와 FDS가 소속되는 Validation 프로젝트의 범위, 검증 방식, 진행률 및 상태 관리 |
| `requirement` | 프로젝트별 URS 요구사항의 항목 번호, 카테고리, 상세 내용, 근거 규정, 개정 버전 및 상태 관리 |
| `fds_spec` | 프로젝트별 FDS 문서의 문서 번호, 제목, 버전, 개정 순번 및 작성·검토·승인 상태 관리 |
| `fds_item` | FDS 문서에 포함되는 기능·화면·인터페이스 항목의 구분, 관리 번호, 기능명, 상세 설명 및 관련 화면 관리 |
| `fds_interface` | FDS 문서의 시스템 간 인터페이스별 송신·수신 시스템, 연동 데이터, 전송 주기, 전송 방식 및 관련 FDS 번호 관리 |

---

## 6. DDS·DQ 관리
<img width="5840" height="4046" alt="image" src="https://github.com/user-attachments/assets/0389d811-f499-4e5f-9db0-530dfa497fce" />
Link : https://drawsql.app/teams/minho-kim/diagrams/06-dds-and-dq-management

### 구조 개요

조직에 소속된 시스템을 기준으로 Validation 프로젝트를 구성하고, URS와 FDS를 기반으로 DDS 상세 설계를 작성한 뒤 설계 적격성을 평가하는 구조입니다.

`dds_spec`과 `dds_item`에서 데이터베이스, 컴포넌트, 인터페이스, 보안 및 배치 등의 상세 설계를 관리합니다. `dq_assessment`와 `dq_item`에서는 URS 요구사항이 FDS와 DDS 설계에 적절하게 반영되었는지 평가하고 그 결과를 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트와 FDS·DDS·DQ 문서 및 상세 항목을 작성·수정·검토하는 사용자 정보 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 식별정보, 관리번호, GAMP 범주, GxP 구분 및 상태 관리 |
| `validation_project` | FDS·DDS·DQ가 소속되는 Validation 프로젝트의 범위, 검증 방식, 진행률 및 상태 관리 |
| `requirement` | DQ 평가 기준이 되는 프로젝트별 URS 요구사항, 개정 버전 및 상태 관리 |
| `fds_spec` | DDS 작성과 DQ 평가의 기반이 되는 FDS 문서의 번호, 버전, 개정 순번 및 상태 관리 |
| `fds_item` | FDS 문서의 기능·화면·인터페이스별 상세 기능과 설계 내용을 관리 |
| `dds_spec` | 프로젝트별 DDS 문서의 문서 번호, 제목, 버전, 개정 순번 및 승인 상태 관리 |
| `dds_item` | DDS 문서에 포함되는 데이터베이스, 컴포넌트, 인터페이스, 보안 및 배치 상세 설계 항목 관리 |
| `dq_assessment` | 프로젝트별 설계 적격성 평가 문서의 번호, 버전, 개정 순번 및 상태 관리 |
| `dq_item` | URS 요구사항과 FDS·DDS 설계의 연계 내용, 적격성 평가 결과, 검토자 및 비고 관리 |

---

## 7. FRA 위험평가 관리
<img width="2575" height="2129" alt="image" src="https://github.com/user-attachments/assets/077a700f-9e8b-483f-ac75-e73a29f8a150" />
Link : https://drawsql.app/teams/minho-kim/diagrams/07-fra-risk-assessment-management

### 구조 개요

조직에 소속된 시스템을 기준으로 Validation 프로젝트를 구성하고, 프로젝트별 URS 요구사항과 연계하여 기능 위험을 평가하는 구조입니다.

`fra_assessment`에서 FRA 문서의 버전과 상태를 관리하고, `fra_item`에서 기능별 위험 시나리오, 제품 영향, 발생 가능성, 탐지 가능성, 위험도 및 완화 전략을 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트와 FRA 문서 및 위험 항목을 작성·수정하는 사용자 정보 관리 |
| `system_asset` | Validation 대상 시스템이나 장비의 식별정보, 관리번호, GAMP 범주, GxP 구분 및 상태 관리 |
| `validation_project` | FRA가 소속되는 Validation 프로젝트의 범위, 검증 방식, 진행률 및 상태 관리 |
| `requirement` | FRA 위험 항목의 평가 기준이 되는 프로젝트별 URS 요구사항과 개정 버전 관리 |
| `fra_assessment` | 프로젝트별 FRA 문서의 번호, 제목, 버전, 개정 순번 및 상태 관리 |
| `fra_item` | 기능별 위험 시나리오, PI·LL·DL 평가값, 위험도, 위험 등급, 완화 전략 및 연결 시험 관리 |

---

## 8. IQ·OQ·PQ 적격성 시험 관리
<img width="3070" height="2543" alt="image" src="https://github.com/user-attachments/assets/dcbb8a59-325e-4e99-9be1-873c70aabc53" />
Link : https://drawsql.app/teams/minho-kim/diagrams/08-iq-oq-and-pq-qualification-testing-management

### 구조 개요

조직에 소속된 시스템과 Validation 프로젝트를 기준으로 IQ·OQ·PQ 프로토콜과 상세 시험 항목을 작성하고, 시험 수행 결과와 일탈을 관리하는 구조입니다.

각 적격성 평가의 문서 헤더와 상세 시험 항목을 분리하여 관리하며, 프로토콜 승인 이후 수행자, 실제 결과, 판정 및 수행 시각을 기록합니다. 시험 수행 중 발생한 문제는 `deviation`에서 조사·해결·종결 상태로 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 적격성 문서를 작성·수정하고 시험을 수행하는 사용자 정보 관리 |
| `system_asset` | IQ·OQ·PQ 수행 대상 시스템이나 장비의 식별정보와 상태 관리 |
| `validation_project` | IQ·OQ·PQ가 소속되는 Validation 프로젝트의 범위, 검증 방식 및 진행 상태 관리 |
| `iq_assessment` | IQ 문서의 번호, 버전, 개정 정보, 프로토콜 상태 및 수행 레코드 상태 관리 |
| `iq_item` | IQ 설치 확인 절차, 기대 결과, 실제 결과, 판정, 수행자 및 수행 시각 관리 |
| `oq_assessment` | OQ 문서의 번호, 버전, 개정 정보, 프로토콜 상태 및 수행 레코드 상태 관리 |
| `oq_item` | OQ 운전 기능 시험 절차, 기대 결과, 실제 결과, 판정, 수행자 및 수행 시각 관리 |
| `pq_assessment` | PQ 문서의 수행계획, 일정, 수행 방식, 버전, 프로토콜 상태 및 수행 레코드 상태 관리 |
| `pq_item` | PQ 성능 시험 절차, 기대 결과, 실제 결과, 판정, 수행자 및 수행 시각 관리 |
| `deviation` | 문서 또는 시험 수행 중 발생한 일탈의 내용, 심각도, 조사·해결 및 종결 승인 상태 관리 |

---

## 9. 설계·위험 산출물 추적성 관리
<img width="4360" height="5206" alt="image" src="https://github.com/user-attachments/assets/a1587eae-c5f3-47d7-9309-7495b8c0044d" />
Link : https://drawsql.app/teams/minho-kim/diagrams/09-design-and-risk-deliverable-traceability-management

### 구조 개요

Validation 프로젝트의 URS 요구사항과 FDS·DDS 설계, DQ 평가 및 FRA 위험평가 산출물 간 추적 관계를 관리하는 구조입니다.

각 문서의 헤더와 상세 항목을 함께 구성하고, `traceability_link`에서 산출물 항목 간 구현, 평가 및 위험 완화 관계를 다형 참조 방식으로 연결합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `app_user` | 설계·위험 산출물과 추적 관계를 작성·수정하는 사용자 정보 관리 |
| `validation_project` | 설계·위험 산출물과 추적 관계가 소속되는 Validation 프로젝트 관리 |
| `requirement` | 추적성의 기준이 되는 프로젝트별 URS 요구사항과 개정 버전 관리 |
| `fds_spec` | 프로젝트별 FDS 문서의 번호, 버전, 개정 정보 및 상태 관리 |
| `fds_item` | URS를 기능·화면·인터페이스 설계로 구현한 FDS 상세 항목 관리 |
| `dds_spec` | 프로젝트별 DDS 문서의 번호, 버전, 개정 정보 및 상태 관리 |
| `dds_item` | 데이터베이스, 컴포넌트, 인터페이스, 보안 및 배치 상세 설계 항목 관리 |
| `dq_assessment` | 프로젝트별 설계 적격성 평가 문서의 버전, 개정 정보 및 상태 관리 |
| `dq_item` | URS와 FDS·DDS 설계 간 연계 내용 및 설계 적격성 평가 결과 관리 |
| `fra_assessment` | 프로젝트별 기능 위험평가 문서의 버전, 개정 정보 및 상태 관리 |
| `fra_item` | URS 또는 기능별 위험 시나리오, 위험도, 위험 등급 및 완화 전략 관리 |
| `traceability_link` | URS, FDS, DDS, DQ, FRA 항목 간 구현·평가·완화 관계를 다형 참조 방식으로 관리 |

---

## 10. 적격성 시험 추적성·RTM 관리
<img width="2840" height="2703" alt="image" src="https://github.com/user-attachments/assets/2173e7a8-503a-42f0-b909-66593b263fa5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/10-qualification-test-traceability-and-rtm-management

### 구조 개요

Validation 프로젝트의 URS 요구사항과 IQ·OQ·PQ 시험 항목 간 추적 관계를 관리하고, 이를 기반으로 요구사항 추적 매트릭스와 산출물 커버리지를 관리하는 구조입니다.

`traceability_link`에서 URS와 적격성 시험 항목 간 검증 관계를 관리하고, `rtm_assessment`와 `rtm_item`에서 프로젝트 및 URS별 추적 결과와 승인 시점의 RTM 스냅샷을 보존합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `app_user` | 적격성 시험, 추적 관계 및 RTM 문서를 작성·수정하는 사용자 정보 관리 |
| `validation_project` | 적격성 시험과 RTM이 소속되는 Validation 프로젝트 관리 |
| `requirement` | 시험 추적성과 RTM의 기준이 되는 프로젝트별 URS 요구사항 관리 |
| `iq_assessment` | 프로젝트별 IQ 문서의 버전, 프로토콜 상태 및 수행 레코드 상태 관리 |
| `iq_item` | URS 검증을 위한 IQ 설치 확인 절차와 시험 결과 관리 |
| `oq_assessment` | 프로젝트별 OQ 문서의 버전, 프로토콜 상태 및 수행 레코드 상태 관리 |
| `oq_item` | URS 검증을 위한 OQ 운전 기능 시험 절차와 시험 결과 관리 |
| `pq_assessment` | 프로젝트별 PQ 수행계획, 문서 버전 및 시험 진행 상태 관리 |
| `pq_item` | 실제 운영 조건에서 수행하는 PQ 성능 시험 절차와 결과 관리 |
| `traceability_link` | URS와 IQ·OQ·PQ 시험 항목 간 검증 관계를 다형 참조 방식으로 관리 |
| `rtm_assessment` | 프로젝트별 URS 건수, FRA 연계율, IQ·OQ 커버리지 및 전체 평균 커버리지의 RTM 스냅샷 관리 |
| `rtm_item` | URS별 FRA·FDS·DDS 매핑, IQ·OQ·PQ 결과 및 항목별 커버리지의 상세 스냅샷 관리 |

---

## 11. VSR·일탈 관리
<img width="4760" height="4306" alt="image" src="https://github.com/user-attachments/assets/d5c6ffd5-1702-4f1a-8de3-8585984fc5b2" />
Link : https://drawsql.app/teams/minho-kim/diagrams/11-vsr-and-deviation-management

### 구조 개요

Validation 프로젝트의 활동 수행 결과, RTM 커버리지 및 일탈 현황을 종합하여 최종 밸리데이션 결론과 VSR을 관리하는 구조입니다.

`project_activity`에서 프로젝트별 활동 진행 상태를 관리하고, `deviation`에서 미해결 일탈 여부를 확인합니다. `vsr_assessment`와 `vsr_item`에서는 프로젝트의 최종 결론과 활동별 문서·시험·일탈·승인 결과를 요약합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 프로젝트 활동, 일탈 및 VSR 문서를 작성·수정·처리하는 사용자 정보 관리 |
| `system_asset` | VSR 작성 대상이 되는 Validation 시스템이나 장비의 식별정보 관리 |
| `validation_project` | 활동, 일탈, RTM 및 VSR이 소속되는 Validation 프로젝트 관리 |
| `validation_activity` | VSR에 집계할 Validation 활동의 코드, 명칭 및 기본 순서 관리 |
| `project_activity` | 프로젝트별 수행 활동의 선택 여부, 필수 여부, 진행 상태 및 승인 시각 관리 |
| `rtm_assessment` | VSR에서 참조하는 프로젝트별 요구사항 추적성과 커버리지 결과 관리 |
| `deviation` | 문서 및 시험 수행 중 발생한 일탈의 심각도, 상태, 해결 내용 및 종결 승인 관리 |
| `vsr_assessment` | 프로젝트별 최종 밸리데이션 결론, 결론 상세 내용, 버전 및 승인 상태 관리 |
| `vsr_item` | Validation 활동별 문서 번호, 개정 차수, 결과 건수, 일탈 및 승인 정보를 요약 관리 |

---

## 12. Workflow·승인·전자서명 관리
<img width="4640" height="3766" alt="image" src="https://github.com/user-attachments/assets/55c8d805-4684-4d27-9288-ad118a65bc37" />
Link : https://drawsql.app/teams/minho-kim/diagrams/12-workflow-approval-and-electronic-signature-management

### 구조 개요

Validation 프로젝트의 문서 상신부터 단계별 검토·승인·반려까지의 Workflow와 전자서명 증적을 관리하는 구조입니다.

프로젝트 참여자와 역할을 기준으로 단계별 담당자를 지정하고, `workflow_instance`, `workflow_step`, `approval_action`을 통해 전체 Workflow와 실제 처리 이력을 관리합니다. 승인·반려 시 생성되는 전자서명은 대상 문서 버전과 내용 해시를 함께 보존합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 시스템과 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 문서 상신자, 단계 담당자, 실제 처리자 및 전자서명자 정보 관리 |
| `role` | 프로젝트 참여자와 Workflow 담당자에게 적용할 역할 기준정보 관리 |
| `system_asset` | 검토·승인 대상 Validation 프로젝트의 시스템이나 장비 정보 관리 |
| `validation_project` | Workflow와 프로젝트 참여자가 소속되는 Validation 프로젝트 관리 |
| `project_member` | 프로젝트별 참여 사용자와 수행 역할, 참여 상태 및 참여 기간 관리 |
| `workflow_instance` | 대상 문서별 전체 검토·승인 Workflow의 상신 정보, 현재 단계 및 진행 상태 관리 |
| `workflow_step` | Workflow 내 단계 순서, 단계 유형, 담당자, 처리 기한 및 단계별 상태 관리 |
| `approval_action` | 상신·검토·승인·반려·취소 등 단계별 실제 처리 내용과 처리자 및 처리 시각 관리 |
| `electronic_signature` | 검토·승인·반려 시 서명자, 서명 의미, 대상 버전, 내용 해시 및 재인증 결과 관리 |

---

## 13. 파일·증적·파일 정리 관리
<img width="2500" height="1791" alt="image" src="https://github.com/user-attachments/assets/be3ac81c-3cb4-46c1-b19e-80c1cc60e362" />
Link : https://drawsql.app/teams/minho-kim/diagrams/13-file-evidence-and-file-cleanup-management

### 구조 개요

Validation 프로젝트에서 사용하는 첨부파일과 시험 증적의 메타데이터 및 업무 대상과의 연결 관계를 관리하고, 임시파일과 만료파일의 정리 작업을 관리하는 구조입니다.

`file_asset`에서 실제 저장 파일의 메타데이터와 정리 상태를 관리하고, `evidence_link`에서 문서·시험 항목·일탈과 증적 파일 간 관계를 관리합니다. `file_cleanup_execution`에서는 정리 대상 조회, 삭제 처리, 실패 및 재시도 이력을 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 파일과 프로젝트 관련 사용자 및 시스템이 소속되는 조직 기준정보 관리 |
| `app_user` | 파일 업로드자와 파일 정리 작업 요청자 및 처리 사용자 정보 관리 |
| `system_asset` | 파일과 증적이 발생하는 Validation 대상 시스템이나 장비 정보 관리 |
| `validation_project` | 증적 파일 연결이 소속되는 Validation 프로젝트 관리 |
| `file_cleanup_execution` | 임시·만료·고립 파일 정리 작업의 실행 방식, 상태, 처리 건수, 실패 및 재시도 이력 관리 |
| `file_asset` | 첨부파일, 증적파일, 리포트 및 내보내기 파일의 저장 경로, 크기, 형식, 만료 및 정리 상태 관리 |
| `evidence_link` | 파일과 문서·시험 항목·일탈 간 N:M 증적 연결을 다형 참조 방식으로 관리 |

---

## 14. 리포트·알림·백업 운영 관리
<img width="2800" height="2423" alt="image" src="https://github.com/user-attachments/assets/5643c343-a8e0-4bf0-a773-d2411268b98b" />
Link : https://drawsql.app/teams/minho-kim/diagrams/14-report-notification-and-backup-operations-management

### 구조 개요

Validation 프로젝트와 Workflow를 기준으로 운영 리포트의 정기·수동 생성, 검토·승인 알림 발송 및 시스템 데이터와 파일의 백업 실행을 관리하는 구조입니다.

`report_schedule`과 `report_generation`에서 리포트 일정과 생성 작업을 관리하고, 생성된 결과 파일은 `file_asset`에 연결합니다. `notification_delivery`에서는 Workflow 관련 알림의 발송과 재시도를 관리하며, `backup_execution`에서는 정기·수동 백업의 실행 상태와 결과를 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 운영 리포트와 사용자가 속하는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 리포트·백업 요청자, 알림 수신자 및 운영 작업 생성·수정자 정보 관리 |
| `system_asset` | 운영 리포트와 백업의 대상이 되는 시스템이나 장비 정보 관리 |
| `validation_project` | 프로젝트 단위 리포트, 알림 및 운영 작업의 기준이 되는 Validation 프로젝트 관리 |
| `workflow_instance` | 알림 대상이 되는 문서별 전체 검토·승인 Workflow 정보 관리 |
| `workflow_step` | 승인 요청, 처리기한 도래 및 지연 알림의 기준이 되는 단계별 담당자와 상태 관리 |
| `file_asset` | 생성된 리포트 결과 파일의 저장 경로, 크기, 형식 및 정리 상태 관리 |
| `report_schedule` | 정기 리포트의 유형, 실행주기, 조회기간, 출력 형식 및 다음 실행 시각 관리 |
| `report_generation` | 리포트 생성 요청, 조회 조건, 실행 상태, 결과 파일, 실패 및 재시도 이력 관리 |
| `notification_delivery` | 검토·승인 요청과 처리기한 관련 알림의 수신자, 채널, 발송 상태, 실패 및 재시도 이력 관리 |
| `backup_execution` | 데이터베이스와 파일의 정기·수동 백업 유형, 대상, 상태, 저장 위치, 크기 및 재시도 이력 관리 |

---

## 15. AI 생성 작업·결과 관리
<img width="2488" height="2229" alt="image" src="https://github.com/user-attachments/assets/52061c1a-be1e-4375-a75b-1bd04c5ad0d5" />
Link : https://drawsql.app/teams/minho-kim/diagrams/15-ai-generation-job-and-result-management

### 구조 개요

Validation 프로젝트의 문서와 항목에 대한 AI 생성 요청을 비동기 작업으로 관리하고, 생성 결과의 선택과 실제 업무 데이터 반영 여부를 관리하는 구조입니다.

`ai_generation_job`에서 AI 모델, 입력 조건, 실행 상태 및 재시도를 관리합니다. `ai_generation_result`와 `ai_result_item`에서는 생성 결과 집합과 상세 항목의 선택·채택 및 적용 대상 정보를 관리합니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | AI 기능을 사용하는 사용자와 대상 시스템이 소속되는 조직 기준정보 관리 |
| `app_user` | AI 생성 요청자, 결과 선택자 및 작업 생성·수정자 정보 관리 |
| `system_asset` | AI 생성 대상 문서가 속하는 Validation 시스템이나 장비 정보 관리 |
| `validation_project` | AI 생성 작업과 대상 업무 데이터가 소속되는 Validation 프로젝트 관리 |
| `ai_generation_job` | AI 생성 작업 유형, 대상 엔터티, 모델, 입력 조건, 실행 상태, 실패 및 재시도 이력 관리 |
| `ai_generation_result` | AI 작업으로 생성된 결과 집합의 제목, 선택 여부, 적용 여부, 선택 사용자 및 선택 시각 관리 |
| `ai_result_item` | AI가 생성한 개별 요구사항, 위험 시나리오, 시험 항목 또는 문서 섹션의 내용과 적용 대상 관리 |

---

## 16. Audit Trail 관리
<img width="1563" height="904" alt="image" src="https://github.com/user-attachments/assets/60be2276-6f0d-407e-987f-4a1f4b0a6fdb" />
Link : https://drawsql.app/teams/minho-kim/diagrams/16-audit-trail-management

### 구조 개요

조직에 소속된 사용자가 수행한 주요 데이터 변경과 시스템·배치 작업의 처리 이력을 감사 목적으로 보존하는 구조입니다.

`audit_trail`에서 작업 유형, 대상 테이블과 레코드, 변경 전후 값, 변경 사유, 수행 주체, 요청·세션 정보 및 대상 문서 버전을 관리합니다. 감사 대상은 다형 참조 방식으로 식별하며, 시스템 또는 배치 작업은 사용자 없이 기록할 수 있습니다.

### 테이블별 역할 요약

| 테이블 | 역할 |
|---|---|
| `organization` | 감사 대상 사용자가 소속되는 고객사 또는 운영 조직의 기준정보 관리 |
| `app_user` | 주요 데이터 변경을 수행한 사용자 계정과 소속 조직 정보 관리 |
| `audit_trail` | 데이터 생성·수정·삭제 및 운영 작업의 수행자, 대상, 변경 전후 값, 변경 사유, 요청·세션 정보와 문서 버전 관리 |
