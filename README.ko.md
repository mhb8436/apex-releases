<!-- 이 파일은 apex-releases 저장소의 루트 README 로 배포된다.
     릴리스 워크플로가 통째로 복사하므로, 배포 repo에서 직접 고치면
     다음 릴리스에 지워진다. 문구 수정은 여기서 한다.
     링크가 docs/ 로 시작하는 것은 배포 repo 루트 기준이기 때문이다. -->
# APEX — 외부 반출이 금지된 코드를 위한 오프라인 정적 분석 도구

> Java, JavaScript, HTML, CSS, SQL, XML을 지원하는 **535개 규칙** 기반 정적 분석 도구.
> 단일 엔진 기반의 데스크톱 앱(GUI)과 CLI, 소스코드 편집기 내장, **완전한 로컬 완결형(Zero-Egress)**.
> 별도 설치 없이 압축 해제 후 즉시 실행 — JVM, Node.js, Python 불필요.

[![Release](https://img.shields.io/github/v/release/mhb8436/apex-releases?label=release&color=2f855a)](https://github.com/mhb8436/apex-releases/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/mhb8436/apex-releases/total?color=2f855a)](https://github.com/mhb8436/apex-releases/releases)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#설치-안내)
[![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-blue)](https://www.gnu.org/licenses/agpl-3.0)

[**다운로드**](https://github.com/mhb8436/apex-releases/releases/latest) ·
[빠른 시작 가이드](docs/QUICK_START.ko.md) ·
[커스텀 규칙 작성법](docs/CUSTOM_RULES.ko.md) ·
[English](README.md)

## 왜 APEX인가

**소스코드의 외부 반출이 원천 차단된 환경에서 작동합니다.**  
국방, 공공, 금융 등 가장 엄격한 보안 심사가 요구되는 곳일수록 대다수의 최신 개발 도구는 반입조차 어렵습니다. APEX는 외부 네트워크 연결이 일체 필요 없으며, `--offline` 옵션을 통해 이를 시스템 레벨에서 강제합니다. DNS 질의를 포함하여 루프백(127.0.0.1) 이외의 모든 아웃바운드 연결을 해석된 IP 레벨에서 원천 차단합니다.  
[외부 통신을 의도적으로 실패시켜 차단 여부를 증명하는 데모](docs/AIRGAP_VERIFICATION.ko.md)를 확인해 보세요.

**별도의 코드 에디터를 반입할 필요가 없습니다.**  
앱 내에서 프로젝트 디렉터리를 탐색하고, 구문 강조(Syntax Highlighting)가 적용된 소스를 읽으며, 결함 위치로 즉시 이동해 수정하고, 해당 파일만 바로 재검사할 수 있습니다. 실행 파일 반입마다 보안 승인이 필요한 현장에서 승인 절차를 하나 줄일 수 있습니다.

**언제나 일관된 검사 결과를 보장합니다.**  
동일한 소스코드라면 언제 돌려도 항상 동일한 순서로 동일한 결함을 검출합니다. 정밀한 분석이 필요한 규칙은 단순 정규식 텍스트 매칭이 아닌 AST(추상 구문 트리) 기반으로 판정하므로 공식 감리 및 감사에서도 높은 신뢰성을 보장합니다.

**사내 인프라에서 구동하는 AI 감리 보고서.**  
사내 구축형 Ollama나 vLLM 서버 등 OpenAI 규격을 지원하는 엔드포인트라면 무엇이든 연동됩니다. 코드가 외부 클라우드 API로 전송될 염려 없이 온프레미스 환경에서 보고서를 작성할 수 있습니다.

**기존 린터가 놓치는 DB 스키마 교차 분석.**  
DDL 스키마 파일만 지정하면 MyBatis XML, 독립 `.sql` 파일, Java 소스 내 인라인 SQL까지 실제 스키마와 대조 분석합니다. 누락된 인덱스, 복합 인덱스 선두 컬럼 누락, 존재하지 않는 컬럼 참조 등을 정확하게 잡아냅니다.

<!-- APEX-DEMO:START -->
## 데모

### 점검 실행 — 데스크톱 앱

![APEX 점검](docs/media/scan.gif)

전자정부 표준프레임워크 기반의 실제 프로젝트(413개 파일, 128,497줄)를 2.56초 만에 점검하는 화면입니다. (경로와 클래스명은 마스킹 처리되었으며, 수치는 실제 측정값입니다.)

### 소스 열람 및 즉시 수정·재검사 — 데스크톱 앱

![APEX 편집기](docs/media/editor.gif)

기존 정적 분석 도구 대부분은 결함 검출 보고서 제공에 그칩니다. 반면 APEX는 파일 검색 후 소스코드를 열면 발견된 결함이 에디터 여백(Gutter)과 미니맵에 시각화되어 방향키나 `F8` 단축키로 빠르게 오갈 수 있습니다.

[편집]을 눌러 코드를 수정하고 저장하면 **해당 파일만** 즉시 증분 재검사합니다. 11건의 결함이 10건으로 줄어드는 것을 확인하기 위해 프로젝트 전체를 다시 돌릴 필요가 없습니다.

에디터가 앱에 내장되어 있어, 모든 실행 파일 반입 시 승인이 필요한 현장에서 반입 승인 대상을 하나 줄일 수 있습니다.

### 커스텀 규칙 추가 — 데스크톱 앱

![커스텀 규칙](docs/media/custom-rule.gif)

팀 개발 표준을 정규식으로 등록하면 다음 점검부터 바로 반영됩니다. 저장 전 샘플 코드를 붙여넣어 매칭 결과를 즉시 검증할 수 있으며, 실제 점검 엔진과 동일한 로직으로 판정하므로 사전 테스트에서 잡힌 패턴이 본 점검에서 누락되는 일은 없습니다.

### 완전 오프라인 환경의 AI 감리 보고서 생성 — 터미널

![폐쇄망 AI 감리 보고서](docs/media/airgap-ai-report.gif)

OpenAI 호환 규격을 지원하는 엔드포인트라면 사내 구축형 Ollama나 vLLM 서버로도 완벽히 작동합니다. 클라우드 API도, 외부 인터넷 연결도 쓰지 않습니다.

핵심은 `--offline` 옵션입니다. 루프백 이외의 모든 연결을 거부하며, 호스트명이 아닌 실제 해석된 IP 레벨에서 판정하므로 도메인 우회나 DNS 질의 시도 자체까지 차단됩니다.

영상에서는 **먼저** 외부 엔드포인트를 지정하여 DNS 조회 단계부터 실패하는 것을 보여줍니다. 그다음 엔드포인트만 로컬 서버로 바꾼 동일한 명령어가 정상적으로 보고서를 생성합니다. 외부 통신이 차단되어야 할 때 확실히 차단됨을 직접 확인하실 수 있습니다.

직접 실행해보기: [폐쇄망 환경 검증 가이드](docs/AIRGAP_VERIFICATION.ko.md)

<!-- APEX-DEMO:END -->

## 하나의 엔진, 두 가지 인터페이스 (GUI & CLI)

APEX는 동일한 룰 엔진 위에 데스크톱 앱(GUI)과 CLI를 함께 제공합니다. 특정 인터페이스의 기능이 제한된 축소판 형태가 아니며, 상황에 맞춰 유연하게 조합해 사용할 수 있습니다.

| 구분 | 데스크톱 앱 (`apex-gui`) | CLI (`apex`) |
|---|---|---|
| 주요 용도 | 코드 탐색 및 결함 조치, 룰셋 튜닝, 산출물 보고서 작성 | CI/CD 파이프라인, 정기 점검, 배치 스크립트 |
| 지원 플랫폼 | Windows x64, macOS (Apple Silicon / Intel) | Windows, macOS, Linux — x64, ARM64, x86 |
| 보고서 형식 | Excel, HTML, JSON, AI 감리 보고서, DOCX | 콘솔 출력, Excel, HTML, JSON, AI 감리 보고서, DOCX |
| 규칙 설정 | GUI 토글, 정규식 즉석 검증, 룰셋 YAML 직접 편집(유효성 검증 포함) | YAML 설정 파일, CLI 옵션 |

두 인터페이스는 단 하나의 파일로 연결됩니다. **데스크톱 앱에서 저장한 규칙 스냅샷이 CLI가 읽는 바로 그 YAML 파일**이므로, 개발자가 검토하고 결정한 기준이 이후 모든 자동화 실행에 그대로 적용됩니다.

