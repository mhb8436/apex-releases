# APEX 사용자 매뉴얼

> **APEX** (Code Quality Analyzer) v1.0.0
> 소스코드 품질 검사 도구

---

## 1. 개요

APEX는 Java, JavaScript, HTML, CSS, SQL, XML 소스코드를 개발 표준에 따라 정적 분석하는 CLI 도구입니다.

**주요 특징**

- 455개 검사 규칙 내장 (402개 활성)
- YAML 설정만으로 규칙 추가/변경 가능 (재빌드 불필요)
- 콘솔, JSON, HTML, Excel 리포트 출력
- 프로파일 기반 규칙 그룹 관리
- Windows, Linux, macOS 지원

---

## 2. 설치

### 2.1 배포 파일 구조

```
apex-windows-amd64/
  apex-windows-amd64.exe        ← 실행 파일
  configs/
    profiles.yaml               ← 프로파일 정의
    rulesets/
      quality.yaml              ← 코드 품질 (121개, 113 활성)
      secure.yaml               ← 시큐어코딩 (110개, 105 활성)
      sql.yaml                  ← SQL 공통 ANSI (88개, 71 활성)
      sql-oracle.yaml           ← SQL Oracle 전용 (19개, 17 활성)
      sql-format.yaml           ← SQL 포맷/스타일 (7개)
      modernize.yaml            ← 레거시 현대화 (50개, 39 활성)
      spring.yaml               ← Spring 표준 (27개, 24 활성)
      egov.yaml                 ← 전자정부 표준 (33개, 26 활성)
```

### 2.2 Windows 설치

1. `apex-windows-amd64.zip` 압축 해제
2. 원하는 경로에 폴더 배치 (예: `C:\tools\apex\`)
3. (선택) 환경 변수 PATH에 추가

```
[시스템 속성] → [환경 변수] → Path → 편집 → C:\tools\apex 추가
```

### 2.3 설치 확인

```cmd
apex-windows-amd64.exe profiles
```

프로파일 목록이 출력되면 정상입니다.

---

## 3. 기본 사용법

### 3.1 소스코드 검사

```cmd
REM 기본 검사 (전체 규칙)
apex-windows-amd64.exe C:\projects\my-app\src

REM 특정 프로파일로 검사
apex-windows-amd64.exe C:\projects\my-app\src --profile=quality
apex-windows-amd64.exe C:\projects\my-app\src --profile=sql
apex-windows-amd64.exe C:\projects\my-app\src --profile=secure

REM 여러 프로파일 조합
apex-windows-amd64.exe C:\projects\my-app\src --profile=quality,secure,sql
```

### 3.2 사용 가능한 프로파일

| 프로파일 | 규칙 수 | 대상 언어 | 설명 |
|----------|---------|-----------|------|
| `quality` | 113 | Java, JS, HTML, CSS | 코드 품질, 명명규칙, 복잡도 |
| `secure` | 105 | Java, JS | 시큐어코딩 가이드 |
| `sql` | 71 | SQL, XML | ANSI SQL 공통 규칙 (DB 불문) |
| `sql-oracle` | 17 | SQL, XML | Oracle 전용 (NVL, ROWNUM, hints 등) |
| `sql-format` | 7 | SQL, XML | SQL 포맷 강제 (대문자, 80자 등) |
| `modernize` | 39 | Java, XML | eGov 3.x→4.x, javax→jakarta |
| `spring` | 24 | Java | Spring/Boot 개발 표준 |
| `egov` | 26 | Java, XML | 전자정부 프레임워크 표준 |

**SQL 프로파일 조합 가이드**

| DB 환경 | 권장 프로파일 | 명령어 |
|---------|-------------|--------|
| MySQL / PostgreSQL / CUBRID | `sql` | `--profile=sql` |
| Oracle | `sql,sql-oracle` | `--profile=sql,sql-oracle` |
| Oracle + 포맷 강제 | `sql-all` (그룹) | `--profile=sql-all` |
| 아무 DB + 포맷 강제 | `sql,sql-format` | `--profile=sql,sql-format` |

**프로파일 그룹**

| 그룹 | 포함 프로파일 | 설명 |
|------|-------------|------|
| `all` | 전체 8개 | 모든 규칙 검사 (Oracle+포맷 포함) |
| `essential` | quality, secure | 필수 품질+보안 |
| `sql-all` | sql, sql-oracle, sql-format | SQL 전체 (Oracle+포맷) |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring | 전자정부 전체 |
| `migration` | modernize, spring | 마이그레이션 점검 |

### 3.3 출력 형식

```cmd
REM 콘솔 출력 (기본)
apex-windows-amd64.exe C:\src --profile=sql

REM HTML 리포트
apex-windows-amd64.exe C:\src --profile=all -o html --output-file=report.html

REM Excel 리포트
apex-windows-amd64.exe C:\src --profile=all -o excel --output-file=report.xlsx

REM JSON 출력
apex-windows-amd64.exe C:\src --profile=all -o json --output-file=report.json
```

**Excel 리포트 시트 구성**

| 시트명 | 내용 |
|--------|------|
| 요약 | 전체 통계 (파일 수, 이슈 수, 심각도별 분포) |
| 이슈목록 | 전체 이슈 상세 (필터, 정렬 가능) |
| 파일별통계 | 파일별 이슈 수 |
| 적용규칙 | 적용된 규칙 목록과 위반 건수 |

### 3.4 심각도 필터

```cmd
REM high 이상만 표시
apex-windows-amd64.exe C:\src --profile=all --min-severity=high

REM critical만 표시
apex-windows-amd64.exe C:\src --profile=all --min-severity=critical
```

심각도 단계: `low` < `medium` < `high` < `critical`

### 3.5 상세 모드

```cmd
apex-windows-amd64.exe C:\src --profile=sql -v
```

로드된 프로파일, 처리 파일 수, 총 이슈 수 등 상세 정보를 표시합니다.

### 3.6 규칙별 요약 테이블 (CI/CD)

```cmd
apex-windows-amd64.exe C:\src --profile=all --summary
```

규칙 ID별 위반 건수를 테이블로 출력합니다. CI/CD 파이프라인에서 빠르게 확인할 때 유용합니다.

### 3.7 특정 규칙 제외

```cmd
REM 특정 규칙 ID를 제외하고 검사 (오버라이드 파일 없이)
apex-windows-amd64.exe C:\src --profile=all --exclude-rule=quality-lg-005,secure-sql-002
```

### 3.8 크로스파일 분석만 실행

```cmd
REM MyBatis 미사용 SQL ID, Mapper-VO 불일치 등 아키텍처 이슈만 확인
apex-windows-amd64.exe C:\src --profile=all --cross-file-only
```

### 3.9 중복 규칙 병합 제어

여러 프로파일을 조합하면 동일한 이슈가 여러 규칙에서 중복 검출될 수 있습니다. 기본적으로 APEX는 같은 파일+라인에서 중복 검출된 이슈를 자동 병합합니다.

```cmd
REM 기본: 자동 병합 (동일 위치 중복 이슈는 최고 심각도 1건만 유지)
apex-windows-amd64.exe C:\src --profile=all

REM 병합 비활성화: 모든 이슈를 개별 표시 (디버깅/규칙 확인 용)
apex-windows-amd64.exe C:\src --profile=all --no-dedup
```

---

## 4. 규칙 오버라이드

현장 환경에 맞게 규칙의 심각도를 변경하거나 특정 규칙을 끌 수 있습니다.

### 4.1 오버라이드 파일 작성

`overrides.yaml` 파일 생성:

```yaml
overrides:
  # 규칙 비활성화
  - id: "sql-fmt-001"
    enabled: false

  # 심각도 변경 (high → low)
  - id: "sql-smell-017"
    severity: "low"

  # 여러 규칙 한번에 비활성화
  - id: "sql-from-001"
    enabled: false
  - id: "sql-dml-001"
    enabled: false
```

### 4.2 오버라이드 적용

```cmd
apex-windows-amd64.exe C:\src --profile=sql --overrides=overrides.yaml
```

---

## 5. 규칙 상세

### 5.1 SQL 프로파일 구조

SQL 규칙은 DB 환경에 따라 3개 프로파일로 분리되어 있습니다:

#### sql (ANSI SQL 공통 — 88개)

DB 벤더에 무관하게 적용되는 공통 SQL 규칙입니다.
MySQL, PostgreSQL, CUBRID, Oracle 등 모든 DB에서 사용할 수 있습니다.

- SELECT *, INSERT 컬럼 누락, UPDATE/DELETE WHERE 누락
- 서브쿼리, 별칭, 조인 조건 검사
- SQL 냄새 (Code Smell): NATURAL JOIN, ON 1=1, AND/OR 괄호 등
- MyBatis: ${} SQL Injection, SELECT *, WHERE 누락

#### sql-oracle (Oracle 전용 — 19개)

Oracle DB 환경에서만 의미 있는 규칙입니다.

| 분류 | 규칙 예시 |
|------|----------|
| Oracle 힌트 | `/*+ HINT */` 무단 사용, LEADING 외 힌트 금지 |
| Oracle 함수 | NVL, TO_CHAR, DECODE, DBMS_RANDOM, INSTR, REGEXP_LIKE |
| Oracle 구문 | ROWNUM 페이징, EXECUTE IMMEDIATE, FOR UPDATE WAIT |

#### sql-format (포맷/스타일 — 7개)

팀별 코딩 스타일에 따라 선택적으로 적용하는 포맷 규칙입니다.

| ID | 규칙명 | 설명 |
|----|--------|------|
| sql-fmt-001 | SQL 키워드 대문자 강제 | SELECT, FROM 등 대문자 |
| sql-fmt-002 | 라인 80자 제한 | 한 줄 80자 초과 감지 |
| sql-fmt-003 | SQL 내 빈줄 금지 | SQL 중간 빈 라인 감지 |
| sql-sel-004 | 한줄 한컬럼 | 컬럼을 한 줄에 하나만 기술 |
| sql-from-005 | 한줄 한테이블 | 테이블을 한 줄에 하나만 기술 |
| sql-where-007 | 한줄 한조건 | 조건을 한 줄에 하나만 기술 |
| sql-sel-006 | 주석 형식 강제 | `/* */` 형식만 허용 |

### 5.2 SQL 냄새(Smell) 규칙

#### 공통 Smell (sql.yaml 내)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-010 | NULL = 비교 | high | `col = NULL`은 항상 UNKNOWN. IS NULL 사용 |
| sql-smell-011 | NATURAL JOIN | high | 컬럼 추가 시 조인 조건이 암묵적 변경 |
| sql-smell-012 | ON 1=1 가짜 조인 | high | Cartesian Product를 조인으로 위장 |
| sql-smell-013 | 자기 자신 별칭 | low | `COL AS COL` 무의미한 별칭 |
| sql-smell-017 | COUNT(DISTINCT) | low | 대량 데이터 정렬/해시 비용 |
| sql-smell-018 | WHERE 1=1 | low | 동적 SQL 흔적, 코드 정리 필요 |
| sql-smell-020 | DISTINCT + GROUP BY | high | GROUP BY만으로 유일성 보장 |
| sql-smell-021 | INSERT INTO ... ORDER BY | high | INSERT에 ORDER BY는 무의미 |
| sql-smell-023 | DISTINCT + JOIN | high | 잘못된 JOIN 중복을 DISTINCT로 숨김 |
| sql-smell-024 | LEFT JOIN + COUNT(*) | high | NULL 행까지 카운트됨 |
| sql-smell-025 | 동일 컬럼 다중 OR | low | IN 절로 변환 권장 |
| sql-smell-026 | 중첩 CTE | high | WITH 안에 WITH, 순차 CTE로 분리 |
| sql-smell-027 | AND/OR 괄호 누락 | high | 우선순위 실수 유발 |

#### Oracle 전용 Smell (sql-oracle.yaml 내)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-003 | WHERE NVL/COALESCE 감싸기 | high | 인덱스 무효화 |
| sql-smell-014 | ORDER BY DBMS_RANDOM | high | 전체 테이블 스캔+정렬 유발 |
| sql-smell-015 | WHERE절 INSTR | high | 문자열 검색 함수로 인덱스 무효화 |
| sql-smell-016 | REGEXP_LIKE | low | 단순 패턴은 LIKE가 효율적 |
| sql-smell-022 | JOIN ON절 NVL/TO_CHAR | high | ON 조건 함수로 인덱스 무효화 |

#### AST 구조 분석 (6개, 공통)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-030 | LEFT JOIN WHERE 필터 | high | LEFT JOIN이 사실상 INNER JOIN |
| sql-smell-031 | LEFT JOIN 뒤 INNER JOIN | high | INNER JOIN이 LEFT JOIN 무효화 |
| sql-smell-032 | HAVING 비집계 조건 | high | WHERE로 이동해야 성능 향상 |
| sql-smell-033 | 미사용 CTE | high | WITH 정의 후 미참조 |
| sql-smell-034 | 서브쿼리 ORDER BY | high | 페이징 없는 정렬은 무의미 |
| sql-smell-035 | 인라인뷰 별칭 누락 | high | 별칭 없으면 참조 불가 |

### 5.2 규칙 타입 요약

| 타입 | 설명 | 수정 범위 |
|------|------|-----------|
| `regex` | 한 줄 패턴 매칭 | YAML만 수정 |
| `regex-multiline` | 여러 줄 패턴 매칭 | YAML만 수정 |
| `ast-filtered-regex` | 주석 오탐 제거 정규식 (ANTLR4 토큰 필터) | YAML만 수정 |
| `ast-method-call` | Java AST 메서드 호출 검사 | YAML만 수정 |
| `ast-annotation` | Java AST 어노테이션 검사 | YAML만 수정 |
| `ast-import` | Java AST import 경로 검사 | YAML만 수정 |
| `ast-variable` | Java AST 변수/필드 검사 | YAML만 수정 |
| `ast-try-catch` | Java AST 예외 처리 검사 | YAML만 수정 |
| `ast-class` | Java AST 클래스 구조 검사 | YAML만 수정 |
| `ast-method` | Java AST 메서드 구조 분석 | YAML만 수정 |
| `ast-dead-code` | Java AST Dead Code 검출 | YAML만 수정 |
| `ast-multi-cud` | 다건 CUD 트랜잭션 검사 | YAML만 수정 |
| `cross-file` | 크로스파일 분석 (MyBatis 미사용 SQL ID 등) | YAML만 수정 |
| `annotation-missing-attr` | Java 어노테이션 속성 검사 | YAML만 수정 |
| `line-length` | 라인 길이 검사 | YAML만 수정 |
| `ast-sql-*` | SQL AST 구조 분석 | Go 코드 + YAML |
| `method-analysis` | Go 내장 분석 로직 | Go 코드 필요 |

### 5.3 규칙 엔진 동작 방식 비교

APEX의 규칙은 입력 데이터와 매칭 방식에 따라 크게 4가지로 분류됩니다.

#### regex (단일 라인 매칭)

```
입력: file.Lines → 한 줄씩 순회
처리: pattern.MatchString(line)
```

- 파일의 각 줄을 **개별적으로** regex 매칭
- `.`는 줄바꿈(`\n`)을 매칭하지 않음
- 가장 빠르고 단순한 방식
- **한계**: 여러 줄에 걸친 패턴 감지 불가, 주석 내 오탐 가능

#### regex-multiline (전체 파일 매칭)

```
입력: strings.Join(file.Lines, "\n") → 파일 전체를 하나의 문자열로 합침
처리: pattern.FindStringMatch(content) → FindNextMatch 반복
```

- 파일의 **모든 줄을 `\n`으로 join**하여 하나의 문자열로 처리
- `.`가 `\n`도 매칭 (Singleline 모드)
- `SELECT\s+\*\s+FROM` 같은 **여러 줄에 걸친 패턴** 감지 가능
- 라인 번호는 매치 위치에서 역산
- **한계**: 주석 내 오탐 가능, 큰 파일에서 상대적으로 느림

#### ast-filtered-regex (주석 오탐 제거 regex)

```
입력: file.Lines → 한 줄씩 순회 (regex와 동일)
추가: ANTLR4 파서로 주석 라인 맵 생성 → 주석 줄 자동 스킵
```

- 기본 동작은 regex와 동일 (한 줄씩 매칭)
- **차이점**: ANTLR4 토큰 스트림으로 **주석 전용 라인을 자동 필터링**
- 예: `// System.out.println 금지` 같은 주석에서 오탐하지 않음
- Java 파일이 아니거나 AST가 없으면 일반 regex와 동일하게 동작

#### AST 쿼리 (ast-method-call, ast-class, ast-annotation 등)

```
입력: ANTLR4 파스 트리 (구조화된 AST)
처리: FindMethodInvocations(), FindAnnotations(), FindClassDeclarationsInfo() 등
     → 구조화된 데이터에서 속성 조건 매칭
```

- **텍스트가 아닌 파싱된 구조체**를 순회
- 메서드 호출, 어노테이션, 클래스, 변수, try-catch 등을 구조적으로 식별
- 컨텍스트 조건 지원: `inside-loop`, `inside-catch`, `method-name:<regex>` 등
- 주석/문자열 내 오탐이 구조적으로 불가능
- **한계**: Java만 지원 (ANTLR4 파서 필요)

#### 비교 요약표

| 항목 | regex | regex-multiline | ast-filtered-regex | AST 쿼리 |
|------|-------|-----------------|-------------------|----------|
| **입력 단위** | 라인 1개씩 | 파일 전체 (join) | 라인 1개씩 | ANTLR4 파스 트리 |
| **매칭 방식** | 텍스트 정규식 | 텍스트 정규식 | 정규식 + 주석 필터 | 구조체 속성 비교 |
| **멀티라인 패턴** | 불가 | **가능** | 불가 | 해당없음 (구조) |
| **주석 오탐 제거** | 없음 | 없음 | **자동 제거** | **구조적 제거** |
| **컨텍스트 조건** | 없음 | 없음 | 없음 | inside-loop, inside-catch 등 |
| **타임아웃** | 100ms | 500ms | 100ms | 없음 (트리 순회) |
| **지원 언어** | 모든 언어 | 모든 언어 | Java (fallback: 전체) | Java 전용 |
| **YAML 설정** | `type: "regex"` | `type: "regex-multiline"` | `type: "ast-filtered-regex"` | `type: "ast-method-call"` 등 |
| **적합한 용도** | 단순 키워드 감지 | 여러 줄 패턴 | 주석 오탐 제거 필요 시 | 메서드/클래스/구조 분석 |

#### 동일 규칙의 타입별 구현 예시

`System.out.println` 사용을 감지하는 규칙을 3가지 방식으로 작성한 예시:

```yaml
# 1) regex — 주석에서도 오탐 가능
- id: "example-regex"
  pattern:
    type: "regex"
    regex: "System\\.out\\.println"

# 2) ast-filtered-regex — 주석 라인 자동 스킵
- id: "example-ast-filtered"
  pattern:
    type: "ast-filtered-regex"
    regex: "System\\.out\\.println"

# 3) ast-method-call — 실제 메서드 호출 노드만 감지 (가장 정확)
- id: "example-ast-call"
  pattern:
    type: "ast-method-call"
    method: "println"
    qualifier: "System\\.out"
```

| 소스코드 | regex | ast-filtered-regex | ast-method-call |
|----------|-------|-------------------|-----------------|
| `System.out.println("hello");` | 검출 | 검출 | 검출 |
| `// System.out.println 금지` | 오탐 | 스킵 | 스킵 |
| `/* System.out.println */` | 오탐 | 스킵 | 스킵 |
| `String s = "System.out.println";` | 오탐 | 오탐 | 스킵 |

> **선택 가이드**: 단순 키워드는 `regex`, 주석 오탐이 문제되면 `ast-filtered-regex`, 구조적 정확성이 필요하면 `ast-*` 타입을 사용하세요. 여러 줄 패턴은 `regex-multiline`만 가능합니다.

---

## 6. 규칙 추가 가이드

### 6.1 Regex 규칙 추가 (가장 간단)

`configs/rulesets/sql.yaml`에 아래 형식으로 추가합니다:

```yaml
- id: "sql-custom-001"
  name: "사용자 정의 규칙"
  severity: "high"           # low, medium, high, critical
  category: "performance"
  description: "이 규칙의 설명"
  enabled: true
  pattern:
    type: "regex"
    regex: "\\bFORBIDDEN_KEYWORD\\b"
    flags: "i"               # i = 대소문자 무시
  custom:
    rule_id: "SQL-CUSTOM-001"
    fix: "수정 방법 안내"
```

**regex 작성 시 주의사항**

- YAML에서 `\`는 `\\`로 이스케이프
- `\b` = 단어 경계, `\s` = 공백, `\w` = 단어 문자
- `(?:...)` = 비캡처 그룹
- `flags: "i"` 를 `custom:` 아래에 넣으면 대소문자 무시

### 6.2 Regex-multiline 규칙 추가

여러 줄에 걸친 패턴을 검사합니다:

```yaml
- id: "sql-custom-002"
  name: "SELECT 후 불필요한 패턴"
  severity: "high"
  category: "performance"
  description: "SELECT 문에서 특정 패턴이 감지되었습니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "\\bSELECT\\b[\\s\\S]*?\\bFORBIDDEN\\b"
    flags: "i"
  custom:
    rule_id: "SQL-CUSTOM-002"
    fix: "수정 방법 안내"
```

**멀티라인 패턴 핵심**

- `[\\s\\S]*?` = 줄바꿈 포함 모든 문자 (비탐욕적)
- `[\\s\\S]*` = 줄바꿈 포함 모든 문자 (탐욕적)
- 패턴이 너무 넓으면 오탐 발생, `*?` (비탐욕적) 사용 권장

### 6.3 규칙 비활성화

YAML에서 `enabled: false`로 변경하거나 오버라이드 파일 사용:

```yaml
# 방법 1: sql.yaml에서 직접 수정
- id: "sql-smell-016"
  enabled: false       # ← 비활성화

# 방법 2: overrides.yaml 사용 (원본 수정 없이)
overrides:
  - id: "sql-smell-016"
    enabled: false
```

### 6.4 새 프로파일 추가

`configs/profiles.yaml`에 추가:

```yaml
profiles:
  # 기존 프로파일...

  my-rules:
    name: "우리 프로젝트 규칙"
    description: "프로젝트 전용 규칙셋"
    ruleset: "rulesets/my-rules.yaml"
    languages:
      - java
      - sql

groups:
  # 기존 그룹...

  my-project:
    name: "우리 프로젝트 전체"
    description: "우리 프로젝트용 전체 검사"
    profiles:
      - quality
      - secure
      - my-rules
```

그런 다음 `configs/rulesets/my-rules.yaml` 파일을 생성합니다.

### 6.5 새 YAML 규칙셋 파일 생성

```yaml
version: "1.0"
profile: "my-rules"

languages:
  - language: java
    rules:
      - id: "my-java-001"
        name: "System.exit 사용 금지"
        severity: "critical"
        category: "restriction"
        description: "System.exit() 호출은 금지됩니다."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bSystem\\.exit\\s*\\("
        custom:
          rule_id: "MY-JAVA-001"
          fix: "System.exit() 대신 예외를 throw하세요"

  - language: sql
    rules:
      - id: "my-sql-001"
        name: "TRUNCATE 사용 금지"
        severity: "critical"
        category: "restriction"
        description: "TRUNCATE TABLE은 금지됩니다."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bTRUNCATE\\s+TABLE\\b"
          flags: "i"
        custom:
          rule_id: "MY-SQL-001"
          fix: "DBA 승인 없이 TRUNCATE를 사용할 수 없습니다"
```

---

## 7. 실전 활용 예시

### 7.1 SQL 파일만 검사

```cmd
apex-windows-amd64.exe C:\projects\sql-files --profile=sql -o excel --output-file=sql_report.xlsx
```

### 7.2 CI/CD 파이프라인 연동

```cmd
REM critical 이슈가 있으면 빌드 실패 (종료 코드 1)
apex-windows-amd64.exe C:\src --profile=essential --min-severity=critical
if %ERRORLEVEL% NEQ 0 (
    echo "Critical 이슈 발견! 빌드 실패"
    exit /b 1
)
```

### 7.3 배치 파일 예시

`run_apex.bat` 파일 생성:

```bat
@echo off
SET APEX_HOME=C:\tools\apex
SET TARGET=%1

if "%TARGET%"=="" (
    echo 사용법: run_apex.bat [검사대상경로]
    exit /b 1
)

echo === APEX 소스코드 품질 검사 ===
echo 대상: %TARGET%
echo.

%APEX_HOME%\apex-windows-amd64.exe %TARGET% ^
    --profile=all ^
    --profiles-file=%APEX_HOME%\configs\profiles.yaml ^
    -o excel ^
    --output-file=apex_report.xlsx ^
    --min-severity=low ^
    -v

echo.
echo 리포트 생성 완료: apex_report.xlsx
pause
```

실행:
```cmd
run_apex.bat C:\projects\my-app\src
```

### 7.4 MyBatis XML 포함 검사

SQL 프로파일은 `.sql` 파일뿐 아니라 MyBatis XML (`*Mapper.xml`) 내부의 SQL도 함께 검사합니다.

```cmd
REM src 아래 .sql 파일 + .xml(MyBatis) 모두 검사
apex-windows-amd64.exe C:\projects\my-app\src --profile=sql
```

---

## 8. 옵션 전체 목록

| 옵션 | 축약 | 기본값 | 설명 |
|------|------|--------|------|
| `--profile` | `-p` | `all` | 프로파일 지정 (쉼표 구분) |
| `--config` | `-c` | - | 설정 파일 직접 지정 (--profile과 동시 사용 불가) |
| `--profiles-file` | - | `configs/profiles.yaml` | 프로파일 정의 파일 경로 |
| `--overrides` | - | - | 규칙 오버라이드 파일 |
| `--output` | `-o` | `console` | 출력 형식 (console/json/html/excel) |
| `--output-file` | - | stdout | 출력 파일 경로 |
| `--min-severity` | `-s` | `low` | 최소 심각도 필터 |
| `--rules` | - | - | 검사할 카테고리 필터 (쉼표 구분) |
| `--exclude-rule` | - | - | 제외할 규칙 ID (쉼표 구분) |
| `--summary` | - | false | 규칙별 요약 테이블 출력 (CI/CD 용) |
| `--cross-file-only` | - | false | 크로스파일 이슈만 표시 (아키텍처 리뷰 모드) |
| `--no-dedup` | - | false | 중복 규칙 자동 병합 비활성화 |
| `--verbose` | `-v` | false | 상세 출력 |

**서브커맨드**

| 커맨드 | 설명 |
|--------|------|
| `profiles` | 사용 가능한 프로파일 목록 표시 |
| `profiles -v` | 프로파일 상세 정보 표시 |

---

## 9. 문제 해결

### 프로파일 로드 실패

```
프로파일 로드 실패: open configs/profiles.yaml: no such file or directory
```

`--profiles-file` 옵션으로 정확한 경로를 지정하세요:

```cmd
apex-windows-amd64.exe C:\src --profiles-file=C:\tools\apex\configs\profiles.yaml --profile=sql
```

### 한글 깨짐 (콘솔 출력)

```cmd
chcp 65001
apex-windows-amd64.exe C:\src --profile=sql
```

또는 HTML/Excel 출력을 사용하면 한글이 정상 표시됩니다.

### 검사 대상 파일이 없음

APEX는 확장자 기반으로 파일을 수집합니다:

| 프로파일 | 검사 확장자 |
|----------|------------|
| quality | `.java`, `.js`, `.html`, `.css` |
| secure | `.java` |
| sql | `.sql`, `.xml` |
| modernize | `.java`, `.xml` |

---

## 부록: 전체 규칙 수 현황

| 프로파일 | 전체 | 활성 | 주요 검사 항목 |
|----------|------|------|---------------|
| quality | 121 | 113 | 명명규칙, 복잡도, 코드스멜, 레이어 구조, 주석 |
| secure | 110 | 105 | SQL Injection, XSS, 암호화, 인증/인가, 에러처리 |
| sql | 88 | 71 | ANSI SQL 공통: SELECT, JOIN, WHERE, DML, 냄새 |
| sql-oracle | 19 | 17 | Oracle 전용: NVL, TO_CHAR, hints, ROWNUM, DECODE |
| sql-format | 7 | 7 | SQL 포맷: 키워드 대문자, 라인 길이, 한줄한컬럼 |
| modernize | 50 | 39 | eGov 마이그레이션, javax→jakarta, deprecated API |
| spring | 27 | 24 | DI 패턴, @Transactional, REST API, 보안 |
| egov | 33 | 26 | 레이어 명명, Service/Mapper, 공통컴포넌트 |
| **합계** | **455** | **402** | |

> **활성 규칙**: `enabled: true`인 규칙만 검사에 적용됩니다. 비활성 규칙은 다른 프로파일과 중복되어 `enabled: false`로 설정된 것입니다.
