# Validation Management Platform 데이터베이스 기술 구성 제안서

- 작성 기준일: **2026-09-18**
- Database 담당: **김민호**
- 설계 기준: [데이터 테이블 정의서](../data-dictionary.md)의 72개 업무 테이블 및 [회사별 Silo DB 검토안](./SILO_DB_ERD_REVIEW.md)
- 제안 범위: 운영 시스템에 사용할 DB 엔진, 버전, 확장 기능, 데이터 기준, 스키마 변경 및 운영 방식

**SQL과 Amazon Aurora PostgreSQL 17 계열을 기본 구성으로 제안한다. 고객사별 전용 Aurora 클러스터에 동일한 업무 테이블 구조를 배치하고, Flyway로 스키마 변경을 관리한다. pgvector는 Aurora를 AI 검색용 벡터 저장소로 사용하기로 결정한 경우에 적용한다.**

회사별 Silo는 확정 전제이며, 아래 세부 구성은 도입 제안이다. 지원 버전은 AWS 공식 문서를 기준으로 확인했으며, 실제 AWS 계정·배포 리전의 생성 가능 여부와 운영 성능은 환경 구성 단계에서 검증한다.

## 1. 개발 기술 구성표 기재안

아래 내용을 기존 표의 Database 행에 옮겨 적을 수 있다.

| 구분 | 기재할 내용 |
|---|---|
| 영역 | Database |
| 담당 | 김민호 |
| 언어 및 Runtime | **SQL / Aurora PostgreSQL 엔진**. DB 함수·트리거가 필요한 경우 PL/pgSQL 사용 |
| Framework·DB | **Amazon Aurora PostgreSQL 17.10, Aurora 패치 17.10.1 도입 제안**. 고객사별 Silo 구성. **pgvector 0.8.2는 RAG 벡터 저장소 채택 시 적용** |
| 추가 항목 | **UTF8 / UTC / Flyway 기반 SQL 마이그레이션 / 자동 백업·시점 복구 35일 / 운영 Writer 1대 + 다른 AZ Reader 1대 / KMS 저장 암호화·TLS 연결 / pg_stat_statements / 배포 리전별 Engine·Patch·Extension 버전 검증** |

Database 영역의 Runtime은 SQL을 실행하는 DB 엔진을 뜻한다. Java·Node.js·Python 버전은 각각의 애플리케이션 또는 배포 도구 실행 환경에서 관리한다. Flyway는 DB 프레임워크가 아니라 테이블·컬럼 변경을 버전별로 적용하는 도구다.

버전 근거: [AWS Aurora PostgreSQL 릴리스 노트](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Updates.html), [AWS 확장 기능 지원표](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Extensions.html).

## 2. 선정 이유

| 현재 시스템의 특성 | 제안 구성과의 연결 |
|---|---|
| 프로젝트·요구사항·시험·결재 등 72개 업무 테이블의 관계 | FK·UNIQUE·CHECK와 트랜잭션을 사용하는 관계형 DB가 적합 |
| UUID, JSONB, timestamptz, 조건부 유일 인덱스를 사용하는 명세 | PostgreSQL을 기준으로 운영 스키마를 구현할 수 있음 |
| 승인 원문·전자서명·감사기록의 보존 | 관련 저장을 하나의 트랜잭션으로 처리하고, 개정·감사 규칙을 구현하는 방향과 일치 |
| 회사별 데이터 격리 | 고객사별 전용 Aurora 클러스터에 같은 스키마를 반복 배치 |
| 고객사별 DB에 동일한 변경 적용 | Flyway 변경 파일과 적용 이력으로 DB별 스키마 버전을 관리 |
| 향후 AI 검색 기능 | 필요한 경우 pgvector를 추가해 벡터 저장·검색을 구성 |

17 계열은 팀의 기존 검토 방향과 현재 명세에 필요한 기능을 기준으로 선정한다. AWS는 18 계열도 제공하므로, 17을 최신 메이저 버전이라는 이유로 추천하는 것은 아니다. 현재 명세에서 18 전용 기능을 필수로 요구하는 근거는 확인되지 않았다. [AWS 제공 버전](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Updates.html)

DB 제품 선정과 별도로, 승인 이후 변경 통제·서명 원문 생성·다형 참조 검증 등의 업무 규칙은 애플리케이션과 운영 스키마에서 구현해야 한다.

## 3. 엔진·버전 제안

| 항목 | 제안값 | 적용 기준 |
|---|---|---|
| DB 서비스 | Amazon Aurora PostgreSQL-Compatible Edition | 고객사별 전용 클러스터 |
| PostgreSQL 엔진 | **17.10** | 대상 리전에서 제공되는지 확인 후 확정 |
| Aurora 패치 | **17.10.1** | 공식 문서에서 확인한 패치 후보. 배포 시 적용된 Aurora 버전을 별도 기록 |
| pgvector | **0.8.2, 조건부 적용** | Aurora 17.10 지원표 기준. AI 검색 저장소와 벡터 모델 설계 확정 후 설치 |
| 엔진 업데이트 | 검증 환경 선행 적용 후 운영 반영 | 보안·안정성 패치와 지원 종료일을 관리하고 고객사별 적용 버전 기록 |

AWS 문서에서 17.10 및 패치 17.10.1의 출시일은 2026-08-21로 확인된다. 확장 지원표의 17.10 열에는 pgvector 0.8.2가 명시되어 있다. **지원되는 버전과 실제 설치된 버전은 구분해 관리한다.** [엔진·패치 릴리스](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Updates.html), [확장 버전](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Extensions.html)

17.10의 표준 지원 종료일은 현재 **2027-12-31**이다. PostgreSQL 17 계열 전체의 지원 기간과 같다고 해석하지 않으며, 운영 계획에 후속 마이너 버전 검증·전환을 포함한다. 같은 마이너 버전의 장기 유지가 우선 요구라면 **17.7 LTS**를 대안으로 검토할 수 있다. 17.7 LTS의 표준 지원 종료일은 현재 2030-02-28이며, 해당 엔진의 pgvector 지원 버전은 0.8.0이다. 본 제안의 기본 후보는 17.10으로 유지한다. [AWS 지원 일정](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/aurorapostgresql-release-calendar.html), [엔진별 확장 조합](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Extensions.html)

## 4. 데이터 저장 기준과 확장 기능

### 4.1 문자셋·시간·데이터 타입

| 항목 | 제안 | 적용 설명 |
|---|---|---|
| DB·클라이언트 문자셋 | **UTF8** | 한글·영문 및 다국어 입력·문서 처리 기준 |
| DB·연결 세션 시간대 | **UTC** | 서명·수행·감사 시각의 저장·전달 기준 통일 |
| 화면 시간대 | 사용자 또는 사업장 설정 적용 | 국내 사업장의 기본 표시값은 Asia/Seoul 제안 |
| 시각 데이터 | `timestamptz` | 시점을 나타내는 작성·수정·서명·수행 시각에 사용 |
| 날짜 데이터 | `date` | 시간대 변환이 필요 없는 날짜만 저장하는 필드에 사용 |
| 식별자·구조화 데이터 | 현 명세의 `uuid`, `jsonb` 유지 | JSON 내부 참조와 서명용 원문의 검증 규칙은 별도로 구현 |
| 문자열 정렬 | Collation을 DB 생성 단계에서 고정 | UTF8과 정렬 규칙은 별개. 한글·영문 혼합 정렬 및 검색을 검증한 뒤 운영 기준 확정 |

UTC 저장·사용자 또는 사업장 시간대 표시는 이미 [데이터 테이블 정의서](../data-dictionary.md)의 기준이다. PostgreSQL의 `timestamptz`는 시점을 내부적으로 UTC로 보관하고 세션 시간대에 따라 표시한다. 사용자가 선택한 시간대 이름 자체를 보존하는 타입은 아니다. [PostgreSQL 문자셋](https://www.postgresql.org/docs/17/multibyte.html), [날짜·시간 타입](https://www.postgresql.org/docs/17/datatype-datetime.html)

### 4.2 Extension 구성

| 확장 기능 | 제안 | 용도·범위 |
|---|---|---|
| `pg_stat_statements` | **기본 적용 제안** | 쿼리별 실행 통계를 수집해 느린 조회와 부하 원인을 분석 |
| `vector` — pgvector | **AI 저장소 결정 후 적용** | 문서·청크 임베딩 저장 및 유사도 검색. 도입 후보는 0.8.2 |
| `pgaudit` | **DB 관리 작업 감사 범위 확정 후 적용** | DDL·권한 변경 등 DB 차원의 작업 기록을 보강. 업무 테이블 `audit_trail`과 함께 사용 |
| 기타 확장 | 실제 사용하는 기능이 있을 때 추가 | 사용 목적·버전·설정·업그레이드 방법을 명시 |

17.10 지원표에는 위 확장들이 포함되어 있다. 확장별 사전 로드 설정과 재시작 필요 여부를 확인하여 파라미터 그룹과 설치 SQL을 함께 관리한다. [AWS 확장 기능 지원표](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Extensions.html)

UUID 생성에 사용하는 `gen_random_uuid()`는 PostgreSQL 17의 내장 함수다. UUID 생성만을 이유로 `pgcrypto`나 `uuid-ossp` 설치를 필수 구성에 추가하지 않는다. [PostgreSQL UUID 함수](https://www.postgresql.org/docs/17/functions-uuid.html)

현재 72개 테이블은 AI 생성 요청·결과·적용 이력을 정의하며, RAG 문서 청크·임베딩 테이블은 아직 정의하지 않는다. pgvector를 채택할 때 AI 담당과 **원문·청크 연결, 임베딩 모델·차원, 검색 인덱스, 고객사별 검색 범위**를 별도로 설계한다. 확장 설치만으로 이 구조가 완성되는 것은 아니다.

## 5. 회사별 Silo와 운영 구성

### 5.1 고객사별 배치

```text
A고객사 → A사 전용 Aurora 클러스터 → A사 업무 DB·동일한 72개 업무 테이블
B고객사 → B사 전용 Aurora 클러스터 → B사 업무 DB·동일한 72개 업무 테이블
```

고객사마다 전용 클러스터, DB 접속 자격증명, 백업·복구 대상을 구분한다. 인증된 고객사 정보로 접속 대상을 결정한다. Silo ERD 검토안에 따라 `organization_id`는 고객사 내부 조직을 식별하는 용도로 유지하는 안을 적용한다. [Silo ERD 검토안](./SILO_DB_ERD_REVIEW.md), [AWS Silo 모델](https://docs.aws.amazon.com/prescriptive-guidance/latest/saas-multitenant-managed-postgresql/silo.html)

라이브러리·규정·Workflow·서명·감사·파일 메타데이터도 기본적으로 해당 고객사 DB에 저장한다. 기본 라이브러리와 공개 규정은 고객사 DB에 적재하여 기존 FK와 승인 당시 근거를 유지한다. 파일 원본은 고객사별 접근을 통제한 객체 저장소에서 관리한다.

### 5.2 가용성·접속·보안

| 항목 | 제안 | 최종 확정에 필요한 사항 |
|---|---|---|
| 운영 가용성 | **Writer 1대 + 다른 AZ의 Reader 1대** | 장애전환·재접속 검증, 고객사별 운영 비용 |
| 컴퓨트 방식 | Provisioned를 초기 운영 검토 기준으로 사용 | 데이터량·동시 사용자·벡터 부하·비용을 측정해 인스턴스 사양 확정. 부하 변동이 크면 Serverless 방식과 비교 |
| DB 네트워크 | 프라이빗 네트워크 배치, 허용된 서버에서만 접속 | 백엔드·AI·배포 작업의 접속 경로 및 보안 그룹 |
| 저장 암호화 | **KMS 기반 암호화 적용** | 키 소유·접근·교체 정책. 고객사별 전용 키 사용 여부는 별도 결정 |
| 통신 암호화 | **TLS 강제 및 서버 인증서·호스트명 검증** | `rds.force_ssl=1`, 클라이언트 `sslmode=verify-full`과 신뢰 CA 구성 |
| 접속 계정 | 앱 실행·스키마 배포·운영 조회 계정 분리 | 필요한 권한만 부여하고 비밀정보 관리 서비스로 자격증명 관리 |
| 접속·조회 | 고객사별 연결 풀 및 접속 한도 관리 | 승인·변경 직후 일관성이 필요한 조회는 Writer에서 처리 |
| 모니터링 | CPU·메모리·연결 수·쿼리 지연·잠금·복구 가능 시점 관찰 | 경보 기준, 로그 조회 권한과 보관 기간 |

Aurora의 여러 AZ에 걸친 스토리지 복제와 장애 시 승격할 Reader 인스턴스는 역할이 다르다. 운영안의 Reader는 장애전환 대상을 확보하기 위한 제안이다. [AWS 고가용성](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)

암호화 구성은 AWS의 저장 암호화와 PostgreSQL TLS 연결 기능을 사용한다. [Aurora 저장 암호화](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Overview.Encryption.html), [Aurora PostgreSQL 보안](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Security.html)

고객사별 클러스터 수에 따라 컴퓨트·백업·운영 비용이 늘어나므로 인프라 담당과 회사당 비용을 산정한다. 테이블 수만으로 인스턴스 크기나 월 비용을 확정하지 않는다.

## 6. 마이그레이션과 백업

### 6.1 Migration Tool: Flyway

**버전이 부여된 SQL 변경 파일을 Flyway로 적용하는 방식을 제안한다.** 고객사별 DB에 동일한 변경 파일을 배포하고, 적용 성공·실패와 현재 스키마 버전을 관리한다. Flyway는 Aurora PostgreSQL을 지원하며 PostgreSQL JDBC 드라이버로 연결한다. [Flyway Aurora PostgreSQL 지원](https://documentation.red-gate.com/flyway/reference/database-driver-reference/aurora-postgresql)

| 항목 | 적용 방안 |
|---|---|
| 최초 스키마 | 72개 업무 테이블 전체 정의를 바탕으로 운영용 초기 마이그레이션 작성 |
| 변경 관리 | 적용 완료한 파일을 덮어쓰지 않고 새 버전의 SQL로 변경 추가 |
| 배포 방식 | CI/CD의 별도 DB 변경 단계에서 고객사별 적용. 여러 앱 인스턴스의 기동에 운영 변경 실행을 분산시키지 않음 |
| 검증 | 빈 DB 생성·기존 버전 업그레이드·데이터 보존·제약조건을 확인하고 `validate`로 변경 파일과 이력 검사 |
| 복구 | 수정 마이그레이션과 백업 복구 절차를 준비. 모든 변경이 자동으로 되돌아간다고 가정하지 않음 |
| 도구 버전 | Flyway·PostgreSQL 모듈·JDBC 드라이버 버전을 함께 고정. Backend의 Java·Spring Boot 조합과 검증 |

현재 14개 ERD SQL에는 같은 참조 테이블이 반복되고 일부 컬럼이 생략되어 있다. **ERD 파일들을 그대로 합쳐 운영 DB에 실행하지 않고, 운영용 SQL을 별도로 작성한다.**

Flyway의 `flyway_schema_history`는 도구의 적용 이력 테이블이다. 72개 업무 테이블과 별도로 관리하며, 전체 물리 테이블 수에는 도구가 추가한 테이블이 포함될 수 있다. `validate`가 수동 변경된 모든 DB 객체까지 검사하는 기능인 것으로 해석하지 않는다. [Flyway 이력 테이블](https://documentation.red-gate.com/fd/flyway-schema-history-table-273973417.html), [Validate 범위](https://documentation.red-gate.com/flyway/reference/commands/validate)

### 6.2 Backup·복구 정책

| 항목 | 제안 |
|---|---|
| 자동 백업 | **보관 기간 35일** |
| 시점 복구 | 보관 범위 내 특정 시각으로 복원하는 **PITR** 사용 |
| 중요 변경 전 | 엔진 업그레이드·위험도가 높은 스키마 변경 전 스냅샷 확보 |
| 장기 보관 | 문서·서명·감사기록 보존 요구에 맞춰 스냅샷 또는 AWS Backup 정책 별도 수립 |
| 파일 보존 | DB 밖의 첨부·증적 원본도 별도 보존·복구 정책 적용 |
| 복구 시험 | 운영 전 및 주요 변경 후 시험하고, 운영 중 정기 주기 수립 |

Aurora는 자동 백업 보관 기간을 **1~35일**로 설정할 수 있다. 위의 35일은 이 시스템에 제안하는 복구용 기간이다. 고객사의 업무 기록 보존기간을 35일로 정한다는 뜻이 아니다. PITR은 새 클러스터로 복구한 후 접속 전환과 데이터 확인을 수행하는 방식이다. [AWS 백업·복구](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html)

복구 시험에는 회사별 복구 대상, 승인·서명·감사 연결, 파일 원본 접근, 최신 스키마 적용 상태를 포함한다. 허용 데이터 손실 범위인 RPO와 서비스 복구 목표 시간인 RTO는 업무 요구에 맞춰 정하고 실제 복구 시험으로 확인한다.

## 7. 최종 확정 항목과 담당 협의

| 확정할 항목 | 제안 기준 | 협의 대상 |
|---|---|---|
| AWS 리전·엔진·패치 | 17.10 / 17.10.1 후보의 리전 가용성 및 실제 적용 버전 확인 | Database·인프라 |
| 인스턴스 사양·비용 | 고객사별 Writer + Reader 운영안으로 부하·비용 측정 | Database·인프라 |
| Flyway·드라이버 버전 | 백엔드와 배포 환경에서 호환성 검증 후 버전 고정 | Database·Backend·CI/CD |
| pgvector 도입 | Aurora를 벡터 저장소로 사용할지 결정하고 모델·차원·스키마 정의 | Database·AI |
| 정렬·검색 기준 | UTF8·UTC를 기본으로 한글·영문 정렬과 검색 결과 확인 | Database·Backend |
| 백업·장기 보관·복구 목표 | 35일 PITR 제안과 별도로 기록 보존기간·RPO·RTO 확정 | Database·인프라·업무 책임자 |
| 고객사 격리 검증 | DB 연결·파일·AI 작업에서 다른 고객사 데이터 접근 차단 확인 | Database·Backend·AI·인프라 |

## 8. 배포 시 버전 확인 방법

아래 명령은 환경 구성 담당자가 실행할 확인 절차이며, 본 제안서 작성 과정에서 운영 AWS 계정이나 DB에 실행한 결과가 아니다. 서울 리전을 사용하는 경우의 예시이며, 배포 리전이 다르면 `ap-northeast-2`를 변경한다.

```powershell
aws rds describe-db-engine-versions --engine aurora-postgresql --db-parameter-group-family aurora-postgresql17 --region ap-northeast-2 --query "DBEngineVersions[].{Version:EngineVersion,Status:Status}" --output table
```

목록에서 생성 가능한 EngineVersion을 확인하고, 실제 생성한 DB에서 엔진·Aurora 패치·설치 확장을 각각 기록한다. [AWS 리전별 버전 조회](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraPostgreSQLReleaseNotes/AuroraPostgreSQL.Updates.html), [엔진·Aurora 버전 식별](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.html)

```sql
SELECT version(), aurora_version();
SHOW server_encoding;
SHOW client_encoding;
SHOW TimeZone;

SELECT name, default_version, installed_version
FROM pg_available_extensions
WHERE name IN ('vector', 'pg_stat_statements', 'pgaudit');

SELECT extname, extversion
FROM pg_extension
WHERE extname IN ('vector', 'pg_stat_statements', 'pgaudit');
```

확장 조회 결과가 없거나 `installed_version`이 NULL이면 설치 완료로 기록하지 않는다. 엔진 업데이트 후에도 실제 설치된 확장 버전을 다시 확인한다.
