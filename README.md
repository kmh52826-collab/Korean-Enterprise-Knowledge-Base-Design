# Validation Management Platform & Hybrid Knowledge Base Design
> **AI-Powered Validation Management Platform for Regulated Environments**

**Status:** Active / In Progress (Start: Aug 2026)  
**Role:** Data Architect (Relational Data Modeling & AI Knowledge Base Data Design)  
**Current Progress:** Phase 1 - Relational Data Modeling (Completed) / Phase 2 - AI Knowledge Base & Vector/Graph DB (In Progress)

> ⚠️ **Notice & Disclaimer:**  
> - **WIP Project Notice:** 본 리포지토리는 현재 개발이 진행 중인(Work-in-Progress) 기업 프로젝트 문서입니다. 시스템 아키텍처 및 상세 모듈 구현이 지속적으로 업데이트되고 있으며, 일부 문서나 코드 섹션은 계속 보완 중(under construction)일 수 있습니다.  
> - **Security & Dummy Data Disclaimer:** 본 리포지토리 및 기술 문서에 포함된 모든 예시 데이터, 수치, 식별자, 소스 코드 샘플 등은 정보 보안 및 기밀 유지를 위해 가공·대체된 **가상 데이터(Dummy Data)** 입니다. 실제 기업 내부의 영업 비밀, 보안 데이터, 실사용 고객 정보는 일절 포함되어 있지 않습니다.

---

## 📌 Project Executive Summary

본 프로젝트는 제약·바이오 분야의 GMP 및 CSV (Computerized System Validation) 규정을 준수하며, 파편화되어 있던 Validation 업무 전 과정을 디지털화하는 **AI 기반 Validation Management Platform** 구축 프로젝트입니다.

검증 계획과 사전 평가부터 URS, 설계(FDS/DDS), 위험평가(FRA), 적격성 시험(IQ/OQ/PQ), 일탈 관리, 종합 보고(VSR)에 이르는 **Validation End-to-End 라이프사이클**을 단일 플랫폼에서 통합 관리하도록 설계되었습니다.

외부 AI 엔지니어들이 벡터 DB와 LLM으로 AI 기능을 개발하고, 저는 플랫폼 전체와 AI가 읽고 쓰는 데이터의 설계를 담당합니다. 본 포트폴리오는 그중 제가 맡은 두 가지 축에 초점을 맞추고 있습니다:
- **규제 준수용 관계형 데이터 모델링 (RDBMS ERD)** `[완료]`
- **AI 감사 대응 및 End-to-End 추적성을 위한 하이브리드 지식 기반 (Vector DB & Graph DB)의 데이터 설계와 준비** `[진행 중]`

---

## 🛠 Tech Stack

| Category | Technologies |
| :--- | :--- |
| **Relational Database** | Amazon RDS (PostgreSQL) |
| **AI & Knowledge Databases** | pgvector, Graph DB (Neo4j) |
| **Object Storage & AI Pipeline** | Amazon S3, Amazon Bedrock API |
| **Languages** | Python, SQL |

---

## 🏗️ System Overview & Architecture

[![System Overview & Context](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)](https://github.com/user-attachments/assets/26fe8971-9772-43d5-a1f0-de782b84f96c)

> 💡 **Tip:** 위 이미지를 클릭하시면 원본 해상도의 더 크고 선명한 전체 다이어그램을 확인하실 수 있습니다.

### 🎯 Architecture Summary
- **Validation Lifecycle** : 검증 계획부터 URS, 설계, FRA, IQ/OQ/PQ, 일탈 관리, VSR까지 규제 환경 (CSV) 전 과정을 종단간 (End-to-End) 지원
- **Core AI Audit & Traceability** : 규제 감사관 (Auditor)의 심층 질의에 대해 AI가 하이브리드 지식 기반을 통해 관련 서류를 탐색·제공하고, End-to-End Lineage 및 승인 이력을 추적할 수 있도록 설계
- **Compliance & Governance** : 21 CFR Part 11 및 ALCOA++ 원칙을 준수하는 데이터베이스 레벨의 Audit Trail 및 이력 관리
- **Hybrid Knowledge Architecture** : 
  - **RDBMS (PostgreSQL)** : Core Data, Audit Trail, Source of Truth 관리
  - **Vector DB (pgvector)** : Semantic Search 및 AI Context Retrieval
  - **Graph DB (Neo4j)** : End-to-End Lineage 및 Impact/Dependency Analysis
  - **Object Storage (S3)** : Document Repository & Evidence Storage

---

## 🚀 Key Technical Contributions & Engineering Design

### 1. End-to-End Relational Data Modeling & ERD Design (Completed)
규제가 엄격한 도메인 (Highly Regulated Domain)의 복잡성을 반영하여, 개념적 모델링부터 논리/물리 ERD 설계까지 14개 업무 영역, 70개가 넘는 테이블의 PostgreSQL 스키마 전체를 직접 설계했습니다. 요구사항이 바뀔 때마다 설계안을 프로젝트 구성원과 PM에게 보고하고 피드백을 반영하며 여러 차례 개정했습니다.

- **Regulatory Compliance Schema Design:** 21 CFR Part 11 (전자서명 및 감사추적) 규정과 ALCOA++ 원칙을 데이터베이스 레벨에서 강제하도록 설계했습니다. 전자서명은 서명한 원문과 대상 개정을 함께 보존하고, 감사기록은 기존 내용을 수정하지 않고 새 기록만 추가합니다.
- **Revision-Pinned Traceability:** 요구사항(URS)과 설계, 위험평가(FRA), 시험 항목(IQ/OQ/PQ) 사이의 추적 관계가 연결 당시의 정확한 개정을 가리키도록 설계했습니다. 새 개정이 생겨도 과거 승인 근거는 바뀌지 않으며, 요구사항은 근거가 된 규정 조항과 그 판본까지 함께 연결됩니다.
- **AI Output Provenance:** AI가 생성한 초안은 곧바로 공식 산출물이 되지 않습니다. 생성, 사용자 선택, 업무 적용을 별도의 상태로 관리하고, 각 항목이 직접 작성된 것인지 AI 초안에서 온 것인지 출처를 기록합니다.
- **Design Trade-off:** 여러 종류의 산출물 개정을 서로 연결하기 위해 다형성 참조(polymorphic reference)를 사용했습니다. 이로 인해 참조 무결성 검증을 데이터베이스 FK가 아닌 애플리케이션 레이어에서 수행해야 하며, 이는 규제 시스템에서 해결해야 할 설계상의 과제로 남아 있습니다.

#### 🔗 [View ERD Specifications](docs/erd-specifications.md)
*(업무 영역별 ERD 설계 문서와 테이블 명세는 위 링크에서 확인하실 수 있습니다.)*

---

### 2. AI Knowledge Base Preparation (In Progress)
AI가 검색하고 문서 초안을 생성하는 데 사용할 지식 기반을 준비하고 있습니다.

- **Document De-identification:** 사내 GMP·CSV 팀이 약 160개 Validation 프로젝트에서 작성한 산출물 문서를 외부 AI 엔지니어가 활용할 수 있도록, 일괄 비식별화하는 처리 로직을 구축하고 있습니다.
- **Vector Indexing (with AI engineers):** 비식별화된 문서를 청크 단위로 파싱·임베딩하여 pgvector에 적재하는 작업을 AI 엔지니어들과 함께 진행하고 있습니다. 이를 통해 단순 키워드 검색을 넘어 문맥 (Semantic) 기반의 컨텍스트 검색이 가능하도록 합니다.

---

### 3. Hybrid Retrieval for Audit Search & Traceability (In Progress)
규제 검토자 (Auditor)의 심층 질의가 들어왔을 때 **AI가 관련 서류를 찾아내고 승인 이력을 추적**할 수 있도록, 관계형 원천 데이터와 연결된 하이브리드 검색 구조를 AI 엔지니어들과 함께 설계하고 있습니다.

- **Graph DB Modeling (Knowledge Graph & Ontology):** 문서 간의 단순 유사도 검색 한계를 극복하기 위해, **"특정 요구사항 (URS)이 어떤 위험평가 (FRA)를 거쳐 최종 승인되었는가"** 에 답할 수 있는 승인 계보 그래프 모델 (Ontology)을 구상하고 있습니다.
- **Validity-Aware Retrieval:** 폐기된 구버전이나 미승인 초안도 질문과 매우 비슷할 수 있지만 감사 근거로는 사용할 수 없습니다. 의미 검색 결과에 관계형 스키마의 개정·승인 정보를 결합해, 현재 유효한 근거만 AI에 제공하는 방안을 설계하고 있습니다.
