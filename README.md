# APEX — 코드 품질 분석 도구

Java, JavaScript, HTML, CSS, SQL, XML 소스코드의 품질/보안/표준 준수를 분석하는 정적 분석 도구입니다.

## 다운로드

[Releases](https://github.com/mhb8436/apex-releases/releases) 탭에서 플랫폼별 바이너리를 다운로드하세요.

| 플랫폼 | 파일 |
|--------|------|
| Windows (64bit) | `apex-windows-amd64.zip` |
| Windows (32bit) | `apex-windows-386.zip` |
| Linux (64bit) | `apex-linux-amd64.tar.gz` |
| Linux (ARM64) | `apex-linux-arm64.tar.gz` |
| macOS (Intel) | `apex-darwin-amd64.tar.gz` |
| macOS (Apple Silicon) | `apex-darwin-arm64.tar.gz` |

## 빠른 시작

```bash
# 압축 해제 후
./apex /path/to/source --profile=all

# 프로파일 지정
./apex /path/to/source --profile=quality,secure

# HTML 리포트
./apex /path/to/source -o html --output-file=report.html
```

## 매뉴얼

- [사용자 매뉴얼](docs/APEX_사용자_매뉴얼.md) — 설치, 실행, 프로파일 설정
- [빠른 시작 가이드](docs/QUICK_START.md)
- [커스텀 규칙 작성](docs/CUSTOM_RULES.md)

## 지원

문의: 관리자에게 연락하세요.
