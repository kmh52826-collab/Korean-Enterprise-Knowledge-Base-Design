# Scalable Data Pipeline & Hybrid Knowledge Base Design for Regulated Environments

**Status:** Active / In Progress (Expected Completion: Dec 2026)  
**Project Start Date:** 2026.08.01  
**Current Phase:** Phase 1 - Relational Data Modeling & Standardization

---

## 1. Project Overview
본 프로젝트는 제약·바이오 산업의 핵심 규제인 GMP 및 CSV(Computerized System Validation) 규정을 준수하는 AI 기반 Validation Management Platform 구축 프로젝트입니다. 기존에 수작업 및 파편화되어 관리되던 URS(요구사항)부터 RA(위험평가), RTM(추적성 매트릭스), IQ/OQ/PQ(적격성평가) 등의 전 Validation 과정을 하나의 플랫폼에서 통합 관리합니다. 궁극적으로 데이터 추적성(Traceability)과 무결성을 보장하는 정형 데이터베이스 구조 위에, 향후 LLM 기반 문서 생성 및 Audit 대응용 지식 구조화(Knowledge Layer)를 결합하는 것을 목표로 합니다.

---

## 2. System Architecture
전체 시스템은 규제 준수(Quality & Compliance)와 AI 지원(AI Engine) 기능이 유기적으로 연결되도록 설계되었습니다.

* **Backend & DB:** Spring Boot / FastAPI 기반 API 구성 및 PostgreSQL을 활용한 정형 데이터(트랜잭션, 감사추적) 영구 저장
* **AI Engine & Knowledge Layer:** Azure OpenAI(LLM)와 Embedding / Vector DB 및 Graph DB를 연계하여 Audit 대응용 RAG 및 하이브리드 지식 베이스(Ontology, LLM Wiki) 구축
* **Storage & Infrastructure:** AWS Cloud 환경에서 S3(Object Storage) 기반 문서 관리 및 CI/CD 파이프라인 구성

---

## 3. Key Technical Contributions

### Phase 1: Complex Relational Data Modeling (Current Focus)
규제가 엄격한 도메인(Highly Regulated Domain)의 특성을 반영하여 데이터 무결성 보장과 완전한 추적성(Traceability) 확보에 집중한 관계형 데이터베이스(PostgreSQL) ERD를 설계했습니다.

* **Regulatory Compliance Data Model:** 21 CFR Part 11(전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 시스템 레벨에서 강제할 수 있도록 Audit Trail 및 Versioning 테이블 구조를 고도화했습니다.
* **End-to-End Traceability (N:M Mapping):** Validation 프로세스 상의 핵심 엔티티(프로젝트 ➔ URS ➔ RA ➔ RTM ➔ Test Protocol) 간의 복잡한 다대다 의존성을 분해하고, 변경 및 승인 이력을 신속히 추적할 수 있는 관계형 스키마를 구현했습니다.

> **[Detail Link] 🔗 [ERD Data Dictionary & Schema Design Document](ERD_LINK_HERE)**  
> *(본 프로젝트의 ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

### Phase 2: Hybrid Knowledge Base & AI Agent Architecture (Proposed)
단순한 관계형 데이터베이스 관리를 넘어, AI 엔진이 Audit 대응 및 단계별 산출물 작성을 지능적으로 지원할 수 있도록 이종 데이터 통합 및 지식 계층(Knowledge Layer)을 설계하고 있습니다.

* **Audit-Ready Semantic Search & Traceability (핵심 기능):**  
  외부 감사(Audit) 대응 시, 규제 검토자가 특정 항목의 승인 여부나 이력을 문의할 때 AI가 수초 내에 관련된 증적 문서, 승인 이력, 연관 요구사항을 정밀하게 추출해 제공하는 RAG 기반 지식 검색 파이프라인을 구축합니다. (Vector DB와 Graph DB를 결합하여 문서의 단순 유사도 검색을 넘어 **승인 관계 및 연관성 추적** 구현)
* **Automated Phase-wise Deliverable Generation:**  
  Validation 단계별(URS, RA, RTM, IQ/OQ/PQ 등) 표준 템플릿과 과거 승인 데이터를 기반으로 규제 준수 가이드라인을 충족하는 산출물 초안(Draft)을 자동 생성하는 LLM 기반 파이프라인을 설계합니다.

---

## 4. Tech Stack
* **Database:** PostgreSQL, Vector DB (Embedding Storage), Graph DB (Ontology / Knowledge Graph)
* **AI / Data Integration:** Azure OpenAI, LangChain / LlamaIndex, RAG Pipeline
* **Backend / Cloud:** Spring Boot, FastAPI, AWS (VPC, ECS/EC2, S3)
* **Compliance Support:** 21 CFR Part 11 (Audit Trail / Electronic Signature)

---

## 5. Open Research Challenges
현재 프로젝트를 진행하며 향후 Ph.D. 과정에서 깊이 있게 탐구하고자 하는 연구 및 기술적 고민입니다.

* **Auditability & Traceability in RAG:** 감사 대응 시 AI가 검색해온 문서와 승인 이력의 정확도를 100% 보장해야 하므로, Vector Search의 환각(Hallucination)을 제어하고 정형 DB의 승인 상태(Approval State)와 비정형 지식을 완벽하게 동기화하는 하이브리드 인덱싱 구조 연구
* **Schema & Knowledge Synchronization:** 트랜잭션 DB(PostgreSQL)의 승인/변경 데이터 발생 시 이를 Graph DB 및 Vector DB에 실시간으로 반영하여 Audit 대응 시 최신 무결성을 유지하는 파이프라인 최적화
