# Scalable Data Pipeline & Hybrid Knowledge Base Design for Regulated Environments

**Status:** Active / In Progress (Expected Completion: Dec 2026)  
**Project Start Date:** 2026.08.01  
**Current Phase:** Phase 1 - Relational Data Modeling & Standardization

---

## 1. Project Overview
본 프로젝트는 제약·바이오 산업의 핵심 규제인 GMP 및 CSV(Computerized System Validation) 규정을 준수하는 AI 기반 Validation Management Platform 구축 프로젝트입니다. 기존에 수작업 및 파편화되어 관리되던 URS(요구사항)부터 RA(위험평가), RTM(추적성 매트릭스), IQ/OQ/PQ(적격성평가) 등의 전 Validation 과정을 하나의 플랫폼에서 통합 관리합니다. 궁극적으로 데이터 추적성(Traceability)과 무결성을 보장하는 정형 데이터베이스 구조 위에, 향후 LLM 기반 문서 생성 및 추천을 위한 지식 구조화(Knowledge Layer)를 결합하는 것을 목표로 합니다.

---

## 2. System Architecture
전체 시스템은 규제 준수(Quality & Compliance)와 AI 지원(AI Engine) 기능이 유기적으로 연결되도록 설계되었습니다.

* **Backend & DB:** Spring Boot / FastAPI 기반 API 구성 및 PostgreSQL을 활용한 정형 데이터(트랜잭션, 감사추적) 영구 저장
* **AI Engine & Knowledge Layer:** Azure OpenAI(LLM)와 Embedding / Vector DB를 연계하여 RAG 및 하이브리드 지식 베이스(Ontology, LLM Wiki) 구축
* **Storage & Infrastructure:** AWS Cloud 환경에서 S3(Object Storage) 기반 문서 관리 및 CI/CD 파이프라인 구성

---

## 3. Key Technical Contributions

### Phase 1: Complex Relational Data Modeling (Current Focus)
규제가 엄격한 도메인(Highly Regulated Domain)의 특성을 반영하여 데이터 무결성 보장과 완전한 추적성(Traceability) 확보에 집중한 관계형 데이터베이스(PostgreSQL) ERD를 설계했습니다.

* **Regulatory Compliance Data Model:** 21 CFR Part 11(전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 시스템 레벨에서 강제할 수 있도록 Audit Trail 및 Versioning 테이블 구조를 고도화했습니다.
* **End-to-End Traceability (N:M Mapping):** Validation 프로세스 상의 핵심 엔티티(프로젝트 ➔ URS ➔ RA ➔ RTM ➔ Test Protocol) 간의 복잡한 다대다 의존성을 분해하고, 변경 발생 시 영향도(Impact)를 추적할 수 있는 관계형 스키마를 구현했습니다.

> **[Detail Link] 🔗 **  
> *(본 프로젝트의 ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

### Phase 2: Hybrid Knowledge Base Architecture (Proposed)
단순한 관계형 데이터베이스 관리를 넘어, AI 엔진이 Validation 업무를 스스로 지원(문서 작성, 추천, 자동 생성)할 수 있도록 이종 데이터 통합 및 지식 계층(Knowledge Layer)을 설계하고 있습니다.

* **Data Standardization for AI:** 과거 Validation 문서(PDF, Word), 템플릿, 가이드라인 및 SOP 등의 비정형 데이터를 수집 및 정제하여 AI가 활용할 수 있도록 메타데이터를 생성하고 표준화합니다.
* **Hybrid Knowledge Architecture:** 요구사항 분석 결과를 바탕으로 RAG(문서 임베딩 및 벡터 검색), 도메인 개념 및 관계를 모델링하는 Ontology, 그리고 LLM Wiki를 결합한 통합 지식 관리 구조를 설계합니다.

---

## 4. Tech Stack
* **Database:** PostgreSQL
* **AI / Data Integration:** Azure OpenAI, Embedding / Vector DB, RAG Pipeline
* **Backend / Cloud:** Spring Boot, FastAPI, AWS (VPC, ECS/EC2, S3)
* **Compliance Support:** 21 CFR Part 11 (Audit Trail / Electronic Signature)

---

## 5. Open Research Challenges
현재 프로젝트를 진행하며 향후 Ph.D. 과정에서 깊이 있게 탐구하고자 하는 연구 및 기술적 고민입니다.

* **Schema Synchronization:** 고도로 정형화된 PostgreSQL 트랜잭션 데이터의 변경 사항을 비정형 Vector DB 및 Ontology 구조(Graph DB)에 실시간으로 무결하게 동기화하는 하이브리드 파이프라인 최적화 방안
* **AI Reliability in Regulated Domains:** 환각(Hallucination)이 치명적인 결과를 낳는 규제 환경에서, RAG 모델의 추론 근거(Reasoning)를 감사(Audit)하고 신뢰성을 검증하기 위한 데이터 아키텍처 설계
