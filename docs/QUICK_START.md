# APEX 현장 배포 퀵스타트 가이드

## 1. 설치 (30초)

### 다운로드

```bash
# macOS (Apple Silicon)
tar -xzf apex-darwin-arm64.tar.gz

# macOS (Intel)
tar -xzf apex-darwin-amd64.tar.gz

# Linux
tar -xzf apex-linux-amd64.tar.gz

# Windows
# apex-windows-amd64.zip 압축 해제
```

### 확인

```bash
./apex --version
```

배포 구조:
```
apex-darwin-arm64          ← 실행 바이너리
configs/
├── profiles.yaml         ← 프로파일 정의
└── rulesets/
    ├── quality.yaml      ← 코드 품질 (53+ 규칙)
    ├── secure.yaml       ← 시큐어코딩 (80+ 규칙)
    ├── sql.yaml          ← SQL 가이드 (89 규칙)
    ├── modernize.yaml    ← 레거시 현대화 (50 규칙)
    ├── spring.yaml       ← Spring 표준 (25 규칙)
    └── egov.yaml         ← 전자정부 표준 (33 규칙)
```

> JVM, Node.js 등 런타임 **불필요**. 바이너리 + configs 폴더만 있으면 동작합니다.

---

## 2. 기본 실행

```bash
# 전체 검사 (6개 프로파일 전부)
./apex /path/to/project --profile=all

# 코드 품질 + 시큐어코딩만
./apex /path/to/project --profile=essential

# 전자정부프레임워크 프로젝트 전용
./apex /path/to/project --profile=egov-full

# 레거시 마이그레이션 진단
./apex /path/to/project --profile=migration
```

### 프로파일 목록

| 프로파일 | 설명 | 규칙 수 |
|----------|------|---------|
| `quality` | 코드 품질, 명명규칙, 성능 | 53+ |
| `secure` | SQL Injection, XSS, 암호화 | 80+ |
| `sql` | SQL 스타일, 바인드변수, DML | 89 |
| `modernize` | eGov/javax/iBatis/Spring 전환 | 50 |
| `spring` | DI, @Transactional, Controller | 25 |
| `egov` | 전자정부 계층/명명/공통 컴포넌트 | 33 |

### 프로파일 그룹

| 그룹 | 포함 | 용도 |
|------|------|------|
| `all` | quality + secure + sql + modernize + spring + egov | 전체 검사 |
| `essential` | quality + secure | 필수 점검 |
| `egov-full` | quality + secure + sql + egov + spring | 전자정부 프로젝트 |
| `migration` | modernize + spring | 레거시 전환 진단 |

---

## 3. 리포트 출력

```bash
# 콘솔 (기본)
./apex /path/to/project --profile=all

# Excel — 기관 제출용
./apex /path/to/project --profile=all -o excel --output-file=report.xlsx

# JSON — CI/자동화 연동
./apex /path/to/project --profile=all -o json --output-file=report.json

# HTML — 웹 공유
./apex /path/to/project --profile=all -o html --output-file=report.html
```

### 심각도 필터

```bash
# high 이상만 출력
./apex /path/to/project --profile=all --min-severity=high

# critical만 출력
./apex /path/to/project --profile=all --min-severity=critical
```

---

## 4. 커스텀 규칙 추가

### 4.1 어디에 추가?

```
configs/rulesets/quality.yaml    ← 기존 파일에 추가 (간단)
configs/rulesets/custom.yaml     ← 새 파일 생성 (규칙이 많을 때)
```

### 4.2 패턴 타입 선택 (의사결정 트리)

```
한 줄에서 찾을 수 있는가?
├── YES → type: "regex"
│
└── NO → 여러 줄에 걸치는가?
    ├── YES → type: "regex-multiline"
    │
    └── Java 구조 분석이 필요한가?
        ├── import 검사 → "ast-import"
        ├── 루프 내 메서드 호출 → "ast-method-call"
        ├── 어노테이션 속성 → "ast-annotation"
        ├── 변수/필드 타입 → "ast-variable"
        ├── try-catch 구조 → "ast-try-catch"
        ├── 클래스 메서드 수 → "ast-class"
        └── 메서드 복잡도/길이 → "ast-method"
```

### 4.3 현장에서 가장 많이 쓰는 5가지

#### (1) 특정 API/메서드 금지

```yaml
- id: "custom-001"
  name: "System.exit() 사용 금지"
  severity: "critical"
  category: "security"
  description: "System.exit()는 서버 프로세스를 종료시킵니다"
  enabled: true
  pattern:
    type: "regex"
    regex: "System\\.exit\\s*\\("
  custom:
    fix: "예외를 throw하거나 Spring의 정상 종료 메커니즘을 사용하세요"
```

#### (2) 금지 라이브러리 import

```yaml
- id: "custom-002"
  name: "프로젝트 금지 라이브러리"
  severity: "high"
  category: "architecture"
  description: "승인되지 않은 라이브러리입니다"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^com\\.alibaba\\.fastjson\\."
  custom:
    fix: "Jackson 또는 Gson을 사용하세요"
```

#### (3) 어노테이션 + 메서드 조합 (다른 줄)

```yaml
- id: "custom-003"
  name: "DELETE 메서드 접두사"
  severity: "medium"
  category: "naming"
  description: "DELETE 메서드는 delete/remove로 시작해야 합니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@DeleteMapping.*\\n\\s*public\\s+\\w+\\s+(?!(delete|remove)\\w+)([a-z]\\w+)\\s*\\("
  custom:
    fix: "delete* 또는 remove* 접두사를 사용하세요"
```

#### (4) 루프 내 위험 호출

```yaml
- id: "custom-004"
  name: "루프 내 DB 조회"
  severity: "high"
  category: "performance"
  description: "루프에서 DB 조회 시 N+1 문제 발생"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(select|find|get|query).*"
    qualifier: ".*(?i)(mapper|dao|repository)"
    context: "inside-loop"
  custom:
    fix: "IN절 배치 조회 또는 JOIN으로 변경하세요"
```

#### (5) 클래스 크기 제한

```yaml
- id: "custom-005"
  name: "Service 클래스 과대"
  severity: "medium"
  category: "design"
  description: "Service 메서드가 30개를 초과합니다"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
  custom:
    max_methods: 30
    fix: "도메인별로 Service를 분리하세요"
```

### 4.4 새 룰셋 파일로 만들기

**① 파일 생성**: `configs/rulesets/custom.yaml`

```yaml
version: "1.0"
profile: "custom"

languages:
  - language: java
    rules:
      - id: "custom-001"
        name: "규칙 이름"
        severity: "high"
        category: "카테고리"
        description: "설명"
        enabled: true
        pattern:
          type: "regex"
          regex: "패턴"
        custom:
          fix: "해결방법"

  - language: xml
    rules:
      - id: "custom-xml-001"
        name: "XML 규칙"
        # ...

  - language: sql
    rules:
      - id: "custom-sql-001"
        name: "SQL 규칙"
        # ...
```

**② 프로파일 등록**: `configs/profiles.yaml`

```yaml
profiles:
  # ... 기존 프로파일 ...

  custom:
    name: "프로젝트 맞춤 규칙"
    description: "프로젝트 고유 코딩 표준"
    ruleset: "rulesets/custom.yaml"
    languages:
      - java
      - xml
      - sql

groups:
  all:
    profiles:
      - quality
      - secure
      - sql
      - modernize
      - spring
      - egov
      - custom          # ← 추가
```

**③ 실행**

```bash
./apex /path/to/project --profile=custom          # 커스텀만
./apex /path/to/project --profile=quality,custom   # 품질 + 커스텀
./apex /path/to/project --profile=all              # 전체 (custom 포함)
```

---

## 5. 규칙 오버라이드

프로젝트별로 기존 규칙의 심각도를 변경하거나 비활성화할 수 있습니다.

**① 오버라이드 파일 생성**: `configs/overrides.yaml`

```yaml
overrides:
  # 심각도 변경
  - rule_id: "quality-nc-001"
    severity: "low"              # high → low로 완화

  # 규칙 비활성화
  - rule_id: "sql-style-001"
    enabled: false               # ANSI JOIN 허용

  # 여러 규칙 일괄 비활성화
  - rule_id: "mod-java-001"
    enabled: false               # new Date() 허용 (레거시 프로젝트)
  - rule_id: "mod-java-003"
    enabled: false               # Calendar 허용
```

**② 실행**

```bash
./apex /path/to/project --profile=all --overrides=configs/overrides.yaml
```

---

## 6. CI/CD 연동

### Jenkins

```groovy
stage('Code Quality') {
    steps {
        sh './apex ./src --profile=all -o json --output-file=apex-report.json'
        sh './apex ./src --profile=all -o excel --output-file=apex-report.xlsx'
        archiveArtifacts artifacts: 'apex-report.*'
    }
}
```

### GitLab CI

```yaml
code-quality:
  stage: test
  script:
    - ./apex ./src --profile=all -o json --output-file=apex-report.json
    - ./apex ./src --profile=all -o excel --output-file=apex-report.xlsx
  artifacts:
    paths:
      - apex-report.*
```

### 품질 게이트 (critical 이슈 시 빌드 실패)

```bash
# APEX는 이슈 발견 시 exit code 1 반환
./apex ./src --profile=essential --min-severity=critical
# exit code 0 = 통과, 1 = critical 이슈 있음
```

---

## 7. 정규식 작성 팁

### YAML 백슬래시 규칙

```yaml
# ✅ 올바른 예 (백슬래시 2개)
regex: "System\\.out\\.print"
regex: "\\bSELECT\\b"
regex: "new\\s+Date\\s*\\("

# ❌ 잘못된 예 (백슬래시 1개)
regex: "System\.out\.print"
regex: "\bSELECT\b"
```

### 자주 쓰는 정규식

| 패턴 | 의미 | 예시 |
|------|------|------|
| `\\.` | 리터럴 점 | `System\\.out` |
| `\\s*` | 공백 0개 이상 | `메서드\\s*\\(` |
| `\\s+` | 공백 1개 이상 | `new\\s+Date` |
| `\\w+` | 단어 문자 1개 이상 | `class\\s+\\w+` |
| `\\b` | 단어 경계 | `\\bSELECT\\b` |
| `[^)]*` | `)` 아닌 모든 문자 | `\\([^)]*\\)` |
| `(?!pattern)` | 부정 전방탐색 | `(?!select)\\w+` |
| `(?i)` | 대소문자 무시 | `(?i)select` |
| `[\\s\\S]*?` | 줄바꿈 포함 (비탐욕) | multiline 전용 |

### regex vs regex-multiline

```yaml
# regex: 한 줄에서 완결
regex: "@Autowired\\s*$"

# regex-multiline: 여러 줄 걸침 (어노테이션 + 선언이 다른 줄)
type: "regex-multiline"
regex: "@Controller[\\s\\S]*?class\\s+\\w+"
```

> **핵심**: `\n` 또는 `[\s\S]`가 패턴에 있으면 반드시 `regex-multiline` 사용. `regex`에서는 절대 매칭 안 됨.

---

## 8. 지원 언어별 패턴 타입

| 언어 | regex | regex-multiline | ast-* (7종) | annotation-missing-attr |
|------|-------|-----------------|-------------|------------------------|
| Java | ✅ | ✅ | ✅ | ✅ |
| JavaScript | ✅ | ✅ | ❌ | ❌ |
| HTML | ✅ | ✅ | ❌ | ❌ |
| CSS | ✅ | ✅ | ❌ | ❌ |
| SQL | ✅ | ✅ | ❌ | ❌ |
| XML | ✅ | ✅ | ❌ | ❌ |
| Properties | ✅ | ✅ | ❌ | ❌ |

> AST 패턴(`ast-*`)은 **Java 전용**입니다. 다른 언어는 regex/regex-multiline을 사용하세요.

---

## 9. 트러블슈팅

### 규칙이 동작하지 않을 때

| 확인 사항 | 해결 |
|-----------|------|
| `enabled: true` 인가? | false면 무시됨 |
| 패턴에 `\n`이 있는데 `type: "regex"` 인가? | `regex-multiline`로 변경 |
| YAML 백슬래시가 `\\.` 가 아니라 `\.` 인가? | `\\.`로 수정 |
| ast-* 규칙인데 Java 파일이 아닌가? | ast-*는 Java 전용 |
| 프로파일에 해당 룰셋이 포함되어 있는가? | profiles.yaml 확인 |

### 너무 많은 이슈가 나올 때

```bash
# 심각도 필터
./apex ./src --profile=all --min-severity=high

# 오버라이드로 특정 규칙 끄기
# configs/overrides.yaml에 enabled: false 추가
./apex ./src --profile=all --overrides=configs/overrides.yaml
```

### JSON으로 규칙별 카운트 확인

```bash
./apex ./src --profile=all -o json --output-file=result.json
python3 -c "
import json
with open('result.json') as f:
    data = json.load(f)
rules = {}
for issue in data.get('issues', []):
    rid = issue.get('rule_id', '')
    rules[rid] = rules.get(rid, 0) + 1
for rid, cnt in sorted(rules.items(), key=lambda x: -x[1]):
    print(f'  {cnt:4d}  {rid}')
print(f'Total: {len(data.get(\"issues\", []))} issues, {len(rules)} rules')
"
```
