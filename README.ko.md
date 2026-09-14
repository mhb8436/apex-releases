<!-- 이 파일은 apex-releases 저장소의 루트 README 로 배포된다.
     릴리스 워크플로가 통째로 복사하므로, 배포 repo에서 직접 고치면
     다음 릴리스에 지워진다. 문구 수정은 여기서 한다.
     링크가 docs/ 로 시작하는 것은 배포 repo 루트 기준이기 때문이다. -->
# APEX — 단일 엔진 기반의 데스크톱 앱 및 CLI 코드 점검 도구

> Java, JavaScript, HTML, CSS, SQL, XML을 **535개 규칙**으로 정밀 점검합니다.
> 별도 설치 과정 없이 데스크톱 앱(GUI)과 CLI 바이너리를 하나의 패키지로 제공합니다.
> JVM, Node.js, Python 등 런타임 의존성이 없으며, 폐쇄망 및 오프라인 환경에서도 즉시 실행됩니다.

[![Release](https://img.shields.io/github/v/release/mhb8436/apex-releases?label=release&color=2f855a)](https://github.com/mhb8436/apex-releases/releases/latest)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#설치)
[![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-blue)](https://www.gnu.org/licenses/agpl-3.0)

> 본 문서는 한국어 안내서입니다. 최신 업데이트는 [영문 README](README.md)를 기준으로 반영됩니다.

[**다운로드**](https://github.com/mhb8436/apex-releases/releases/latest) ·
[빠른 시작 가이드](docs/QUICK_START.ko.md) ·
[English](README.md)

<!-- APEX-DEMO-KO:START -->
## 주요 기능 데모

### 초고속 점검 실행 — 데스크톱 앱

![APEX 점검](docs/media/scan.gif)

전자정부 표준프레임워크 기반의 실제 프로젝트(413개 파일, 128,497라인)를 2.56초 만에 점검하는 화면입니다. (경로와 클래스명은 마스킹 처리되었으며, 점검 수치는 실제 측정치입니다.)

### 소스 코드 열람 및 즉시 수정·재검사 — 데스크톱 앱

![APEX 편집기](docs/media/editor.gif)

기존 정적 분석 도구 대부분은 결함 검출 보고서를 제공하는 단계에서 그칩니다. 반면 APEX는 파일 검색 후 소스코드를 열면 발견된 결함이 에디터 여백(Gutter)과 미니맵에 시각화되어 방향키나 `F8` 단축키로 결함 위치를 빠르게 탐색할 수 있습니다.

[편집] 모드에서 코드를 수정하고 저장하면, 전체 프로젝트를 다시 돌릴 필요 없이 **해당 파일만 즉시 증분 재검사**하여 결함 해결 여부를 실시간으로 반영합니다.

전용 에디터를 앱 내에 탑재하고 있어, 외부 프로그램 반입 및 실행 승인 절차가 까다로운 폐쇄망/공공 사업 현장에서도 별도의 IDE 없이 편리하게 수정 작업을 진행할 수 있습니다.

### 커스텀 규칙 등록 및 즉시 검증 — 데스크톱 앱

![커스텀 규칙](docs/media/custom-rule.gif)

프로젝트 고유의 코딩 컨벤션이나 보안 규약을 정규식(Regex)으로 등록하면 차기 점검부터 즉시 반영됩니다. 저장 전 샘플 코드를 입력하여 탐지 결과를 실시간으로 검증할 수 있으며, 실제 점검 엔진과 동일한 로직으로 판정하므로 사전 검증된 패턴은 본 점검에서도 정확히 탐지됩니다.

### 완전 오프라인 환경의 AI 감리 보고서 생성 — CLI

![폐쇄망 AI 감리 보고서](docs/media/airgap-ai-report.gif)

OpenAI 규격을 지원하는 엔드포인트라면 무엇이든 연동할 수 있습니다. 사내에 구축된 Ollama나 vLLM 프라이빗 서버와 직접 연동되므로, 외부 클라우드 API 호출이나 인터넷 연결이 일체 발생하지 않습니다.

핵심은 `--offline` 옵션입니다. 루프백(127.0.0.1) 이외의 아웃바운드 트래픽을 원천 차단하며, 호스트명이 아닌 실제 해석된 IP 레벨에서 검증하므로 도메인 우회나 DNS 질의(Lookup) 시도 자체까지 철저히 차단됩니다.

데모 영상에서는 **먼저** 외부 엔드포인트를 지정하여 DNS 질의 단계부터 안전하게 차단되는 과정을 보여줍니다. 이후 엔드포인트만 로컬 서버로 변경한 동일 명령어로 오프라인 상태에서 정상적으로 보고서가 생성되는 것을 입증합니다. 외부 통신이 차단되어야 할 때 확실하게 차단됨을 직접 확인하실 수 있습니다.

직접 검증해보기: [폐쇄망 환경 검증 가이드](docs/AIRGAP_VERIFICATION.ko.md)

<!-- APEX-DEMO-KO:END -->

## 하나의 코어 엔진, 두 가지 인터페이스 (GUI & CLI)

APEX는 단일 룰 엔진을 기반으로 데스크톱 앱(GUI)과 명령행 인터페이스(CLI)를 모두 제공합니다. 특정 인터페이스의 기능이 제한된 축소판 형태가 아니며, 사용 환경과 목적에 맞춰 상호보완적으로 활용할 수 있습니다.

| 구분 | 데스크톱 앱 (`apex-gui`) | CLI (`apex`) |
|---|---|---|
| 주요 활용 환경 | 코드 탐색 및 결함 수정, 룰셋 튜닝, 납품용 보고서 생성 | CI/CD 파이프라인 연동, 정기 점검, 배치 스크립트 |
| 지원 플랫폼 | Windows x64, macOS (Apple Silicon / Intel) | Windows, macOS, Linux (x64, ARM64, x86) |
| 보고서 출력 | Excel, HTML, JSON, AI 감리, DOCX | 콘솔 표준 출력, Excel, HTML, JSON, AI 감리, DOCX |
| 규칙 설정 | GUI 토글, 정규식 즉시 검증, 룰셋 YAML 직접 편집(유효성 검사 지원) | YAML 설정 파일, CLI 실행 옵션 |

두 인터페이스는 단일 설정 파일로 긴밀하게 연결됩니다. **데스크톱 앱에서 설정·저장한 규칙 스냅샷(YAML)을 CLI에서도 그대로 참조**하므로, 개발자가 검토하고 확정한 점검 기준이 CI/CD 자동화 환경에서도 100% 동일하게 유지됩니다.

