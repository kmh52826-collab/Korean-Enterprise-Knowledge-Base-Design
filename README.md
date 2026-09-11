# Validation Management Platform & Hybrid Knowledge Base Design
> **AI-Powered Validation Management Platform for Regulated Environments**

**Status:** Active / In Progress (Start: Aug 2026)  
**Role:** Data Architect & Knowledge Engineer (Relational Data Modeling & AI Knowledge Architecture)  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Pipeline & Vector/Graph DB Architecture (In Progress)

> ⚠️ **Notice & Disclaimer:**  
> - **WIP Project Notice:** 본 리포지토리는 현재 개발이 진행 중인(Work-in-Progress) 기업 프로젝트 문서입니다. 시스템 아키텍처 및 상세 모듈 구현이 지속적으로 업데이트되고 있으며, 일부 문서나 코드 섹션은 계속 보완 중(under construction)일 수 있습니다.  
> - **Security & Dummy Data Disclaimer:** 본 리포지토리 및 기술 문서에 포함된 모든 예시 데이터, 수치, 식별자, 소스 코드 샘플 등은 정보 보안 및 기밀 유지를 위해 가공·대체된 **가상 데이터(Dummy Data)** 입니다. 실제 기업 내부의 영업 비밀, 보안 데이터, 실사용 고객 정보는 일절 포함되어 있지 않습니다.

---

## 📌 Project Executive Summary

본 프로젝트는 제약·바이오 분야의 GMP 및 CSV (Computerized System Validation) 규정을 준수하며, 파편화되어 있던 Validation 업무 전 과정을 디지털화하는 **AI 기반 Validation Management Platform (DVT)** 구축 프로젝트입니다.

프로젝트 생성부터 URS, RA, RTM, 적격성평가 (IQ/OQ/PQ), 시험 수행, VSR, 운영 및 변경관리에 이르는 **Validation End-to-End 라이프사이클**을 단일 플랫폼에서 통합 관리하도록 설계되었습니다.

본 포트폴리오는 플랫폼의 핵심 기반이 되는 두 가지 축에 초점을 맞추고 있습니다:
- **규제 준수용 관계형 데이터 모델링 (RDBMS ERD)** `[완료]`
- **AI 기반 심층 감사 대응 및 End-to-End 추적성을 위한 하이브리드 지식 DB (Vector DB & Graph DB) 파이프라인 구축** `[진행 중]`

---

## 🛠 Tech Stack

| Category | Technologies |
| :--- | :--- |
| **Relational Database** | Amazon RDS (PostgreSQL) |
| **AI & Knowledge Databases** | Vector DB, Graph DB (Neo4j / Amazon Neptune) |
| **Object Storage & AI Pipeline** | Amazon S3, Amazon Bedrock API |
| **Languages** | Python, SQL |

---

## 🏗️ System Overview & Architecture

[![System Overview & Context](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)

> 💡 **Tip:** 위 이미지를 클릭하시면 원본 해상도의 더 크고 선명한 전체 다이어그램을 확인하실 수 있습니다.

### 🎯 Architecture Summary
- **Validation Lifecycle** : Project Initiation부터 DQ/FRA, IQ/OQ/PQ, RTM, VSR까지 규제 환경 (CSV) 전 과정을 종단간 (End-to-End) 지원
- **Core AI Audit & Traceability** : 규제 감사관 (Auditor)의 심층 질의에 대해 AI가 하이브리드 지식 DB를 기반으로 관련 서류를 탐색·제공하고, End-to-End Lineage 및 승인 이력을 추적할 수 있도록 설계
- **Compliance & Governance** : 21 CFR Part 11 및 ALCOA++ 원칙을 준수하는 데이터베이스 레벨의 Audit Trail 및 이력 관리
- **Hybrid Knowledge Architecture** : 
  - **RDBMS (PostgreSQL)** : Core Data, Audit Trail, Source of Truth 관리
  - **Vector DB** : Semantic Search 및 AI Context Retrieval
  - **Graph DB** : End-to-End Lineage 및 Impact/Dependency Analysis
  - **Object Storage (S3)** : Document Repository & Evidence Storage

---

## 🚀 Key Technical Contributions & Engineering Design

### 1. End-to-End Relational Data Modeling & ERD Design (Completed)
규제가 엄격한 도메인 (Highly Regulated Domain)의 복잡성을 반영하여, 개념적 모델링부터 논리/물리 ERD 설계까지 전체 PostgreSQL 스키마를 직접 구축했습니다.

- **Regulatory Compliance Schema Design:** 21 CFR Part 11 (전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 데이터베이스 레벨에서 엄격히 강제하기 위해 Audit Trail 및 Versioning 전용 테이블 구조와 이력 추적 아키텍처를 설계했습니다.
- **Complex Traceability Mapping & Normalization:** Validation 프로세스 상의 핵심 엔티티 (프로젝트 -> URS -> RA -> RTM -> Test Protocol) 간의 복잡한 다대다 (N:M) 의존성을 정규화하고, 엔티티 간 영향도 (Impact Analysis)를 정밀하게 추적할 수 있는 스키마를 정의했습니다.

#### 🔗 [View ERD Data Dictionary & Schema Design Document](docs/erd-specifications.md)
*(ERD 상세 설계 문서 및 핵심 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

---

### 2. AI Deliverable Draft Generation Pipeline (Vector & Graph-Augmented Retrieval) (In Progress)
기존에 파일 형태로 파편화되어 있는 사내 규정, 가이드라인, 그리고 이전 프로젝트의 산출물 원본 데이터 (S3 적재)를 활용해, 규제 준수 가이드라인에 부합하는 문서 초안 (Draft)을 자동으로 생성하는 AI 파이프라인을 구축하고 있습니다.

- **Document Ingestion & Vector Embedding:** Amazon S3에 파일 형태로 보관된 비정형 Validation 문서 및 양식들을 청크 (Chunk) 단위로 파싱하고 임베딩하여 **Vector DB**에 적재하는 파이프라인을 설계하고 있습니다. 이를 통해 단순 키워드 검색을 넘어 문맥 (Semantic) 기반의 정밀한 컨텍스트 검색이 가능하도록 설계했습니다.
- **Graph-Augmented Context Integration:** 단순 텍스트 유사도를 넘어, URS와 RA 등 Validation 단계별 **문서 간의 상하관계 및 승인 이력 (Graph DB)** 과 **Vector DB의 유사도 검색 (Semantic Search)** 을 연계하여, 규정 정합성과 연계성이 반영된 고품질의 산출물 초안 (Multi-candidate Drafts)을 LLM 연동을 통해 도출하는 파이프라인을 구현 중입니다.

---

### 3. Hybrid Database Architecture for Advanced Audit Search & Traceability (In Progress)
단순한 문서 초안 생성을 넘어, 향후 규제 검토자 (Auditor)의 심층 질의와 검증 요구가 들어왔을 때 **AI가 하이브리드 지식 DB를 통해 관련 서류를 직접 찾아내고 승인 이력을 추적**할 수 있도록 데이터베이스 구조를 설계 및 구현하고 있습니다.

- **Graph DB Modeling (Knowledge Graph & Ontology):** 문서 간의 단순 유사도 검색 한계를 극복하기 위해, **"특정 요구사항 (URS)이 어떤 위험평가 (RA)를 거쳐 최종 승인되었는가"** 에 대한 객체 간의 관계와 승인 이력 그래프 모델 (Ontology)을 구상했습니다. 이를 통해 감사관 질의 시 승인 관계망을 정밀 추적 (Lineage Tracking)할 수 있는 DB 스키마를 정의했습니다.
- **Hybrid Retrieval Pipeline:** Vector DB의 **Semantic Search (의미 기반 서류 탐색)** 와 Graph DB의 **Structural Lineage (구조적 이력 추적)** 를 결합하여, 감사관의 까다로운 질의에 대해 **AI가 연관 근거 서류와 승인 프로세스를 즉시 도출하여 제공**하는 하이브리드 검색 파이프라인을 구축하고 있습니다.
