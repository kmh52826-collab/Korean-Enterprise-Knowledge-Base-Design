# Scalable Data Pipeline & Hybrid Knowledge Base Design for Regulated Environments

**Status:** Active / In Progress  
**Project Start Date:** Aug 2026  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Pipeline & Vector/Graph DB Architecture (In Progress)  
**Role:** Data Architect & Knowledge Engineer (Relational Data Modeling & AI Knowledge Architecture)

> 💡 **Notice:** This repository documents an active work-in-progress (WIP) enterprise project. As the system is currently under active development, documentation and module implementations are continuously being populated, and certain sections may be under construction.

---

## 1. Project Overview
본 프로젝트는 제약·바이오 산업의 핵심 규제인 GMP 및 CSV(Computerized System Validation) 규정을 준수하는 AI 기반 Validation Management Platform 구축 프로젝트입니다. URS(요구사항), RA(위험평가), RTM(추적성 매트릭스), IQ/OQ/PQ(적격성평가) 등 수작업과 문서로 파편화되어 관리되던 전 Validation 과정을 하나의 플랫폼에서 통합 관리합니다.

본 포트폴리오는 해당 플랫폼의 핵심 기반이 되는 **(1) 규제 준수용 관계형 데이터 모델링(RDBMS ERD, 완료)**과 **(2) 산출물 초안 자동 생성 및 AI 감사 대응을 위한 하이브리드 지식 DB(Vector DB & Graph DB) 데이터 파이프라인 구축(진행 중)**에 초점을 맞추고 있습니다.

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

> 🔗 **[ERD Data Dictionary & Schema Design Document](docs/erd-specifications.md)**
> *(ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

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
