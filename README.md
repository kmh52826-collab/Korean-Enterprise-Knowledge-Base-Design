# Scalable Data Pipeline & Hybrid Knowledge Base Design for Regulated Environments

**Status:** Active / In Progress (Expected Completion: Dec 2026)  
**Project Start Date:** 2026.08.01  
**Current Phase:** Phase 1 - Relational Data Modeling & Standardization

---

## 1. Project Overview
본 프로젝트는 제약·바이오 산업의 핵심 규제인 GMP 및 CSV(Computerized System Validation) 규정을 준수하는 AI 기반 Validation Management Platform 구축 프로젝트입니다. URS(요구사항), RA(위험평가), RTM(추적성 매트릭스), IQ/OQ/PQ(적격성평가) 등 수작업과 문서로 파편화되어 관리되던 전 Validation 과정을 하나의 플랫폼에서 통합 관리합니다.

본 포트폴리오는 해당 플랫폼의 핵심 기반이 되는 **(1) 규제 준수용 관계형 데이터 모델링(RDBMS ERD)**과 **(2) AI 감사 대응 및 문서 생성을 위한 하이브리드 지식 DB(Vector DB & Graph DB) 아키텍처 설계**에 초점을 맞추고 있습니다.

---

## 2. System Overview & Context
*(본인이 설계한 데이터 레이어가 AWS 클라우드 아키텍처 상에서 동작하는 전체 시스템 구성입니다.)*

* **Access & Security:** Route 53, WAF, CloudFront, ALB (보안 및 트래픽 분산)
* **Application Layer:** ECS Fargate 기반 컨테이너 환경 (Web Portal, Backend API, Validation Workflow Engine)
* **Data & Knowledge Layer (Main Role):** 
  * **Relational Data & Cache:** Amazon RDS (PostgreSQL) + ElastiCache (Redis)
  * **File & Knowledge Storage:** Amazon S3 (문서/첨부파일) + Vector Storage & Knowledge Graph (RAG/AI 연계)
* **AI Engine Integration:** AI Orchestrator, RAG / Document Generation Service, LLM API (Azure OpenAI / Bedrock)
* **DevOps & Monitoring:** AWS CodePipeline (CI/CD), CloudWatch, Secrets Manager

---

## 3. My Technical Contributions (Core Focus)

### Contribution 1. Relational Data Modeling & ERD Design (Amazon RDS PostgreSQL)
규제가 엄격한 도메인(Highly Regulated Domain)의 특성을 반영하여, 데이터 무결성 보장과 완전한 추적성(Traceability) 확보에 집중한 PostgreSQL 스키마를 설계했습니다.

* **Regulatory Compliance Schema:** 21 CFR Part 11(전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 데이터베이스 레벨에서 강제할 수 있도록 Audit Trail 및 Versioning 전용 테이블 구조를 구축했습니다.
* **Complex Traceability Mapping (N:M):** Validation 프로세스 상의 핵심 엔티티(프로젝트 ➔ URS ➔ RA ➔ RTM ➔ Test Protocol) 간의 복잡한 다대다 의존성을 정교하게 분해하고, 변경 발생 시 영향도(Impact) 분석이 가능하도록 스키마를 최적화했습니다.

> **[Detail Link] 🔗 [ERD Data Dictionary & Schema Design Document](ERD_LINK_HERE)**  
> *(본 프로젝트의 ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

---

### Contribution 2. Hybrid Database Architecture for AI (Vector DB & Graph DB)
단순한 관계형 데이터 관리를 넘어, AI Orchestrator가 Audit(감사) 대응과 단계별 산출물 작성(Drafting)을 정밀하게 수행할 수 있도록 **Vector DB와 Graph DB를 연계한 지식 데이터베이스(Knowledge Database) 구조**를 설계 및 구축하고 있습니다.

* **Graph DB (Knowledge Graph & Ontology):**  
  문서 간의 단순 유사도를 넘어, **"특정 요구사항(URS)이 어떤 위험평가(RA)를 거쳐 최종 승인되었는가"**에 대한 객체 간의 관계와 승인 이력 그래프를 모델링합니다. 이를 통해 감사관(Auditor) 질의 시 승인 관계망을 완벽하게 추적(Lineage Tracking)할 수 있는 기반을 제공합니다.
* **Vector DB (Semantic Search & RAG):**  
  Amazon S3에 저장된 비정형 Validation 문서(PDF, Word), 표준 작업 지침서(SOP), 규제 가이드라인을 청킹(Chunking) 및 임베딩하여 Vector DB에 저장하고, 유사도 기반의 신속한 세맨틱 검색(Semantic Search)을 지원합니다.
* **Integrated Hybrid Retrieval:**  
  Vector DB의 '의미 기반 검색'과 Graph DB의 '관계/승인 이력 추적'을 결합한 하이브리드 인덱싱 구조를 설계하여, AI 문서 초안 생성 시 환각(Hallucination)을 극소화하고 정밀한 근거 문서를 제시합니다.

---

## 4. Database & Infrastructure Tech Stack
* **Relational Database & Cache:** Amazon RDS (PostgreSQL), Amazon ElastiCache (Redis)
* **AI & Knowledge Databases:** Vector DB (e.g., Pinecone / Qdrant / Pgvector), Graph DB (e.g., Neo4j / Amazon Neptune)
* **Object Storage:** Amazon S3
* **Cloud Infrastructure Context:** AWS (VPC, ECS Fargate, CodePipeline, CloudWatch)

---

## 5. Open Research Challenges
현재 데이터 아키텍처를 설계하며 향후 Ph.D. 과정에서 깊이 있게 탐구하고자 하는 연구 주제입니다.

* **Dual-Database Synchronization:** Amazon RDS PostgreSQL 트랜잭션 DB의 승인/변경 상태를 실시간으로 Graph DB 및 Vector DB에 레이턴시 없이 무결하게 동기화하는 데이터 파이프라인 최적화
* **Auditability in Hybrid DB:** AI 검색 결과가 규제 감사를 통과할 수 있도록, Vector Search와 Graph Traversal을 혼합한 검색 결과의 추론 과정(Reasoning Path)을 역추적할 수 있는 DB 구조 연구
