# Scalable Data Pipeline & Hybrid Knowledge Base Design for Regulated Environments

**Status:** Active / In Progress  
**Project Start Date:** Aug 2026  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Pipeline & Vector/Graph DB Architecture (In Progress)  
**Role:** Data Architect & Knowledge Engineer (Relational Data Modeling & AI Knowledge Architecture)

> ⚠️ **Notice & Disclaimer:**  
> - **WIP Project Notice:** 본 리포지토리는 현재 개발이 진행 중인(Work-In-Progress) 기업 프로젝트 문서입니다. 시스템 아키텍처 및 상세 모듈 구현이 지속적으로 업데이트되고 있으며, 일부 문서나 코드 섹션은 계속 보완 중(under construction)일 수 있습니다.  
> - **Security & Dummy Data Disclaimer:** 본 리포지토리 및 기술 문서에 포함된 모든 예시 데이터, 수치, 식별자, 소스 코드 샘플 등은 정보 보안 및 기밀 유지를 위해 가공·대체된 **가상 데이터(Dummy Data)** 입니다. 실제 기업 내부의 영업 비밀, 보안 데이터, 실사용 고객 정보는 일절 포함되어 있지 않습니다.

---
## 1. Project Overview

본 프로젝트는 제약·바이오 분야의 GMP 및 CSV(Computerized System Validation) 규정을 준수하며, 파편화되어 있던 Validation 업무 전 과정을 디지털화하는 **AI 기반 Validation Management Platform(DVT)** 구축 프로젝트입니다.

프로젝트 생성부터 URS, RA, RTM, 적격성평가(IQ/OQ/PQ), 시험 수행, VSR, 운영 및 변경관리에 이르는 **Validation End-to-End 라이프사이클**을 단일 플랫폼에서 통합 관리하도록 설계되었습니다.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/d6dc32d6-d414-4174-bfb8-d7358823542b" />
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/81f022c2-133b-4aa9-a08c-33c768f18d81" />

본 포트폴리오는 플랫폼의 핵심 기반이 되는 두 가지 축에 초점을 맞추고 있습니다:

* **규제 준수용 관계형 데이터 모델링 (RDBMS ERD) [완료]**
* **AI 기반 감사 대응 및 산출물 초안 자동 생성용 하이브리드 지식 DB(Vector DB & Graph DB) 파이프라인 구축 [진행 중]**

---

## 2. System Overview & Context
*(설계한 데이터 레이어가 AWS 클라우드 아키텍처 상에서 동작하는 전체 시스템 구성입니다.)*

* **Access & Security:** Route 53, WAF, CloudFront, ALB (보안 및 트래픽 분산)
* **Application Layer:** ECS Fargate 기반 컨테이너 환경 (Web Portal, Backend API, Validation Workflow Engine)
* **Data & Knowledge Layer:** 
  * **Relational Data & Cache (Completed):** Amazon RDS (PostgreSQL) + ElastiCache (Redis)
  * **File Storage & Knowledge Pipeline (In Progress):** Amazon S3 (문서/첨부파일) + Vector Storage & Knowledge Graph (RAG/AI 연계)
* **AI Engine Integration:** AI Orchestrator, RAG / Document Generation Service, LLM API (Azure OpenAI / Bedrock)
* **DevOps & Monitoring:** AWS CodePipeline (CI/CD), CloudWatch, Secrets Manager

---

## 3. Key Technical Contributions

### Contribution 1. End-to-End Relational Data Modeling & ERD Design (Completed)
규제가 엄격한 도메인(Highly Regulated Domain)의 복잡성을 반영하여, 개념적 모델링부터 논리/물리 ERD 설계까지 전체 PostgreSQL 스키마를 구축했습니다.

* **Regulatory Compliance Schema:** 21 CFR Part 11(전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 데이터베이스 레벨에서 강제하기 위해 Audit Trail 및 Versioning 전용 테이블 구조와 이력 추적 구조를 설계했습니다.
* **Complex Traceability Mapping & Normalization:** Validation 프로세스 상의 핵심 엔티티(프로젝트 ➔ URS ➔ RA ➔ RTM ➔ Test Protocol) 간의 복잡한 다대다(N:M) 의존성을 정규화하고, 엔티티 간 영향도(Impact Analysis) 추적 스키마를 정의했습니다.

🔗 **[ERD Data Dictionary & Schema Design Document](docs/erd-specifications.md)**  
*(ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

---

### Contribution 2. AI Deliverable Draft Generation Pipeline (In Progress)
Validation 각 단계별(URS, RA, RTM 등) 서식과 표준 데이터를 바탕으로 규제 준수 가이드라인을 충족하는 산출물 초안(Draft)을 자동 생성하는 AI 프롬프트 및 데이터 입출력 파이프라인을 구축하고 있습니다.

* **Template & Data Standardization:** 비정형 문서 템플릿과 표준 가이드라인을 구조화된 메타데이터로 변환하여 LLM 기반 생성 엔진이 일관된 문서 산출물을 생성할 수 있도록 데이터 입출력 파이프라인을 설계했습니다.

---

### Contribution 3. Hybrid Database Architecture for Advanced Audit Search (In Progress)
단순한 문서 생성을 넘어, 향후 규제 검토자(Auditor)의 질의에 신속히 대응할 수 있도록 Vector DB와 Graph DB를 연계한 지식 데이터베이스(Knowledge Database) 구조를 설계 및 구현하고 있습니다.

* **Graph DB Modeling (Knowledge Graph & Ontology):**  
  문서 간의 단순 유사도를 넘어, **"특정 요구사항(URS)이 어떤 위험평가(RA)를 거쳐 최종 승인되었는가"**에 대한 객체 간의 관계와 승인 이력 그래프 모델(Ontology)을 구상했습니다. 이를 통해 감사관 질의 시 승인 관계망을 추적(Lineage Tracking)할 수 있는 DB 스키마를 정의 중입니다.
* **Vector DB & Hybrid Retrieval:**  
  Amazon S3에 저장될 Validation 규제 가이드라인과 SOP를 임베딩하여 Vector DB에 연동하고, Graph DB의 '승인 이력 추적'과 결합해 환각(Hallucination) 없는 고품질 감사 대응 검색 파이프라인을 구축하고 있습니다.

---

## 4. Database & Infrastructure Tech Stack
* **Relational Database & Cache (Implemented):** Amazon RDS (PostgreSQL), Amazon ElastiCache (Redis)
* **AI & Knowledge Databases (In Progress):** Vector DB (e.g., Pinecone / Qdrant / Pgvector), Graph DB (e.g., Neo4j / Amazon Neptune)
* **Object Storage & AI Pipeline:** Amazon S3, Azure OpenAI / Bedrock API
* **Cloud Infrastructure Context:** AWS (VPC, ECS Fargate, CodePipeline, CloudWatch)
