# APEX — Static Code Quality Analyzer

> **500+ rules** for Java, JavaScript, HTML, CSS, SQL, XML. Single binary, zero dependencies, works offline.

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
[![Go](https://img.shields.io/badge/Go-1.24+-00ADD8.svg)](https://go.dev)
[![Platforms](https://img.shields.io/badge/Platforms-Windows%20|%20Linux%20|%20macOS-lightgrey.svg)](#download)

[한국어](#한국어)

## Why APEX?

- **Single binary, zero runtime** — No JVM, Node.js, or Python required. One 15MB executable.
- **Offline / air-gapped** — Works perfectly in restricted environments with no internet access.
- **Deterministic analysis** — AST-based analysis with 100% reproducible results. Zero false positives by design.
- **500+ built-in rules** — Quality, security, SQL, Spring, modernization, and more.
- **DDL × Query cross-analysis** — Validates index usage, query correctness against DDL schema.

## Download

Download from [Releases](https://github.com/mhb8436/apex-releases/releases):

| Platform | Binary |
|----------|--------|
| macOS Apple Silicon | `apex-darwin-arm64.tar.gz` |
| macOS Intel | `apex-darwin-amd64.tar.gz` |
| Linux x86_64 | `apex-linux-amd64.tar.gz` |
| Linux ARM64 | `apex-linux-arm64.tar.gz` |
| Windows x64 | `apex-windows-amd64.zip` |

### Docker

```bash
docker run --rm -v $(pwd):/src ghcr.io/mhb8436/apex /src --profile=essential
```

## Quick Start

```bash
# Download and extract
curl -fsSL https://github.com/mhb8436/apex-releases/releases/latest/download/apex-darwin-arm64.tar.gz | tar xz

# Analyze your project
./apex ./src --profile=essential

# Full scan with all rules
./apex ./src --profile=all
```

## Usage

```bash
# Basic scan
apex ./src --profile=quality

# Multiple profiles
apex ./src --profile=quality,secure

# JSON output
apex ./src --profile=all -o json --output-file=report.json

# HTML report
apex ./src --profile=all -o html --output-file=report.html

# Excel report
apex ./src --profile=all -o excel --output-file=report.xlsx

# Filter by severity
apex ./src --profile=all --min-severity=high

# Korean output
apex ./src --profile=all --lang=ko

# DDL × Query cross-analysis (index validation)
apex ./src --profile=ddl --ddl=/path/to/ddl/
apex ./src --profile=sql-ddl --ddl=schema.sql
```

## Profiles

Profiles group rules by purpose. Combine them as needed.

| Profile | Description | Rules |
|---------|-------------|-------|
| `quality` | Code quality (naming, logging, complexity) | 114 |
| `secure` | Secure coding (injection, crypto, auth) | 84 |
| `sql` | ANSI SQL standards (vendor-independent) | 88 |
| `sql-oracle` | Oracle-specific (NVL, hints, ROWNUM) | 19 |
| `sql-format` | SQL formatting/style | 7 |
| `modernize` | Legacy modernization | 50 |
| `spring` | Spring/Boot standards | 27 |
| `egov` | eGovernment Framework standards | 33 |
| `ddl` | DDL × Query cross-analysis (requires `--ddl`) | 17 |

### Profile Groups

| Group | Includes |
|-------|----------|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov |
| `essential` | quality, secure |
| `sql-all` | sql, sql-oracle, sql-format |
| `sql-ddl` | sql, sql-oracle, ddl |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring |
| `migration` | modernize, spring |

## CLI Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--profile` | `-p` | Profile(s) to use (comma-separated) | - |
| `--config` | `-c` | Config file path | - |
| `--output` | `-o` | Output format (console/json/html/excel) | console |
| `--output-file` | | Output file path | stdout |
| `--min-severity` | `-s` | Minimum severity (low/medium/high/critical) | low |
| `--ddl` | | DDL file/directory path (for index validation) | - |
| `--lang` | | Output language (en/ko) | en |
| `--rules` | | Rule categories to check | - |
| `--exclude-rule` | | Rule IDs to exclude | - |
| `--cross-file-only` | | Cross-file analysis only | false |
| `--summary` | | Rule summary table (CI/CD) | false |
| `--verbose` | `-v` | Verbose output | false |

## Rules Overview

### Code Quality (`quality`)
- Class naming patterns (Controller, Service, Mapper, VO, Util)
- `System.out.println` prohibited — use Logback
- Logger string concatenation prohibited — use `{}` placeholders
- Empty catch blocks, dead code, god classes
- Method length, cyclomatic complexity

### Secure Coding (`secure`)
- SQL Injection: `createStatement()` prohibited, use PreparedStatement
- XSS: `innerHTML`, `document.write` caution
- Command Injection: `Runtime.exec()`, `ProcessBuilder` with external input
- Path Traversal, SSRF, CSRF
- Weak crypto (DES, MD5, SHA-1, RC4) prohibited — use AES, SHA-256+
- Hardcoded passwords/keys prohibited

### SQL (`sql`)
- `SELECT *` prohibited — specify columns
- Leading wildcard in `LIKE` prohibited
- Bind variables required
- `UPDATE`/`DELETE` require `WHERE` clause
- Code smells: `NATURAL JOIN`, `ON 1=1`, missing parentheses

### DDL × Query Analysis (`ddl`)

Requires `--ddl` flag pointing to DDL files. Cross-analyzes DDL schema with SQL queries (MyBatis XML, `.sql` files) to detect:

- **Query correctness** — References to non-existent tables/columns, JOIN type mismatches, INSERT missing NOT NULL columns
- **Index usage** — WHERE/JOIN columns without indexes, composite index leading column skip, function wrapping invalidating indexes (TO_CHAR, NVL)
- **DDL improvements** — Frequently queried columns without indexes, unused/duplicate indexes, FK without indexes, missing primary keys

```bash
# Analyze queries against DDL schema
apex ./src --profile=ddl --ddl=./ddl/schema.sql

# Combine with SQL rules
apex ./src --profile=sql-ddl --ddl=./ddl/
```

## GitHub Action

```yaml
- uses: mhb8436/apex-ai@v1
  with:
    path: './src'
    profile: 'essential'
    min-severity: 'medium'
    lang: 'en'
```

## Documentation

- [User Manual](docs/APEX_사용자_매뉴얼.md)
- [Quick Start Guide](docs/QUICK_START.md)
- [Custom Rules Guide](docs/CUSTOM_RULES.md)

## License

This project is dual-licensed:

- **AGPL-3.0** — Free for open-source use.
- **Commercial License** — For proprietary/commercial use.

---

# 한국어

## APEX — 코드 품질 분석 도구

> **500+ 규칙**으로 Java, JavaScript, HTML, CSS, SQL, XML 소스코드의 품질/보안/표준 준수를 분석합니다.

### 핵심 가치

- **설치 없는 단일 바이너리**: JVM/Node.js 등 런타임 설치 불필요. 15MB 단일 실행 파일
- **폐쇄망(망분리) 환경 지원**: 인터넷 차단 환경에서도 완벽 동작
- **한국형 전자정부 표준 특화**: eGovFrame 아키텍처와 한국 SI 프로젝트 명명 규칙 기본 탑재
- **확정적(Deterministic) 분석**: AST 기반 정밀 분석, 100% 재현 가능한 결과
- **DDL × 쿼리 교차 분석**: DDL 스키마와 소스코드 SQL을 교차 분석하여 인덱스/쿼리 문제 탐지

### 빠른 시작

```bash
# 기본 스캔
./apex /path/to/source --profile=all

# 프로파일 지정
./apex /path/to/source --profile=quality,secure

# HTML 리포트
./apex /path/to/source -o html --output-file=report.html

# DDL × 쿼리 교차 분석 (인덱스 검증)
./apex /path/to/source --profile=ddl --ddl=/path/to/ddl/
./apex /path/to/source --profile=sql-ddl --ddl=schema.sql
```

### 프로파일

| 프로파일 | 설명 | 규칙 수 |
|---------|------|--------|
| `quality` | 코드 품질 점검 | 114 |
| `secure` | 시큐어코딩 가이드 | 84 |
| `sql` | ANSI SQL 공통 규칙 (DB 불문) | 88 |
| `sql-oracle` | Oracle 전용 (NVL, hints, ROWNUM 등) | 19 |
| `sql-format` | SQL 포맷/스타일 (팀별 선택) | 7 |
| `modernize` | 레거시 현대화 | 50 |
| `spring` | Spring/Boot 개발 표준 | 27 |
| `egov` | 전자정부 프레임워크 표준 | 33 |
| `ddl` | DDL × 쿼리 교차 분석 (`--ddl` 필수) | 17 |

### 프로파일 그룹

| 그룹 | 포함 프로파일 |
|------|------------|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov |
| `essential` | quality, secure |
| `sql-all` | sql, sql-oracle, sql-format |
| `sql-ddl` | sql, sql-oracle, ddl |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring |
| `migration` | modernize, spring |

### DDL × 쿼리 교차 분석

`--ddl` 플래그로 DDL 파일을 지정하면, DDL 스키마와 소스코드의 SQL 쿼리(MyBatis XML, .sql 파일)를 교차 분석합니다.

**쿼리 정합성 (ddl-query):**
- DDL에 없는 테이블/컬럼 참조 탐지
- JOIN 컬럼 타입 불일치 (암시적 변환)
- INSERT에 NOT NULL 컬럼 누락
- UPDATE SET에 PK 컬럼 포함

**인덱스 활용 분석 (ddl-index):**
- WHERE/JOIN/GROUP BY/ORDER BY 컬럼 인덱스 누락
- 복합 인덱스 선두 컬럼 스킵
- 함수 래핑 인덱스 무효화 (TO_CHAR, NVL 등)

**DDL 개선 제안 (ddl-improve):**
- 자주 쿼리되는 컬럼에 인덱스 미존재
- FK 컬럼 인덱스 없음
- 미사용/중복 인덱스
- PK 없는 테이블, 과다 인덱스

### CLI 옵션

| 옵션 | 축약 | 설명 | 기본값 |
|-----|------|------|-------|
| `--profile` | `-p` | 사용할 프로파일 (쉼표로 구분) | - |
| `--config` | `-c` | 설정 파일 경로 | - |
| `--output` | `-o` | 출력 형식 (console/json/html/excel) | console |
| `--output-file` | - | 출력 파일 경로 | stdout |
| `--min-severity` | `-s` | 최소 심각도 (low/medium/high/critical) | low |
| `--ddl` | - | DDL 파일/디렉토리 경로 (인덱스 검증용) | - |
| `--lang` | - | 출력 언어 (en/ko) | en |
| `--rules` | - | 점검할 규칙 카테고리 (쉼표로 구분) | - |
| `--exclude-rule` | - | 제외할 규칙 ID (쉼표로 구분) | - |
| `--cross-file-only` | - | 크로스파일 분석만 실행 | false |
| `--summary` | - | 규칙별 요약 테이블 (CI/CD 용) | false |
| `--verbose` | `-v` | 상세 출력 | false |

### 매뉴얼

- [사용자 매뉴얼](docs/APEX_사용자_매뉴얼.md) — 설치, 실행, 프로파일 설정
- [빠른 시작 가이드](docs/QUICK_START.md)
- [커스텀 규칙 작성](docs/CUSTOM_RULES.md)

### 문의

관리자에게 연락하세요.
