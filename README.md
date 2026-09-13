<!-- 이 파일은 apex-releases 저장소의 루트 README 로 배포된다.
     릴리스 워크플로가 통째로 복사하므로, 배포 repo에서 직접 고치면
     다음 릴리스에 지워진다. 문구 수정은 여기서 한다.
     링크가 docs/ 로 시작하는 것은 배포 repo 루트 기준이기 때문이다. -->
# APEX — Code quality analysis, as a desktop app or a CLI

> **535 rules** for Java, JavaScript, HTML, CSS, SQL and XML. One download gives
> you both a desktop app and a command-line binary. No JVM, no Node.js, no
> Python. Runs with no internet access.

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Platforms](https://img.shields.io/badge/Platforms-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](#install)

[한국어 문서](README.ko.md)

<!-- APEX-DEMO:START -->
## Demo

### Running a scan — desktop app

![APEX scan](docs/media/scan.gif)

A real eGovFrame-based project — 413 files, 128,497 lines — analyzed in 2.56
seconds. Paths and class names are anonymized; every number is an actual result.

### Adding a custom rule — desktop app

![Custom rule](docs/media/custom-rule.gif)

Register a team convention as a regex and it is picked up on the next scan. The
pattern is tested against sample code before you save it.

### AI audit report with no internet access — terminal

![Air-gapped AI report](docs/media/airgap-ai-report.gif)

APEX generates the audit report through any OpenAI-compatible endpoint, so an
in-house Ollama or vLLM server works. No cloud API, no outbound connection.

`--offline` is the part that matters. It rejects every connection whose
destination is not loopback — checked at the resolved IP, so a hostname cannot
get around it, and name resolution itself is blocked too.

The recording shows the command pointed at an external endpoint **first**. It
fails, at the DNS lookup. Then the same command with only the endpoint changed
to the local server produces the report. A demo that merely works proves
nothing; a demo that fails when it should is the evidence.

Run the same two steps yourself: [Air-gap verification](docs/AIRGAP_VERIFICATION.md).

<!-- APEX-DEMO:END -->

## Two front ends, one engine

APEX is a desktop app and a CLI over the same rule engine. Neither is a
cut-down version of the other, and you are not meant to pick one forever.

|  | Desktop app (`apex-gui`) | CLI (`apex`) |
|---|---|---|
| Suited to | exploring a codebase, deciding which rules apply, producing a report to hand over | CI pipelines, scheduled scans, scripted batches |
| Platforms | Windows x64, macOS (Apple Silicon / Intel) | Windows, macOS, Linux — x64, ARM64, x86 |
| Reports | Excel, HTML, JSON, AI audit report, DOCX | Console, Excel, HTML, JSON, AI audit report, DOCX |
| Rule tuning | click a toggle, test a regex live | YAML file, CLI flags |

The two are joined by one file. **A rule selection saved in the desktop app is
the same YAML the CLI reads**, so a decision made once by a person is what
every later automated run executes:

```
Desktop app · Rules tab          ruleset.yaml            CI · every build
  toggle rules, set severities  ──▶  save snapshot  ──▶  apex ./src --overrides ruleset.yaml
```

That is the intended division of labour: a person decides in the GUI, the
pipeline repeats in the CLI, and the result is identical because it is the same
engine reading the same file.

## Install

Download from [Releases](https://github.com/mhb8436/apex-releases/releases).
Each bundle is self-contained — unzip and run, nothing to install.

| Platform | Bundle | Contains |
|---|---|---|
| Windows x64 | `apex-windows-amd64.zip` | `apex.exe`, `apex-gui.exe`, `apex-report.exe` |
| macOS Apple Silicon | `apex-darwin-arm64.tar.gz` | `apex`, `apex-gui`, `apex-report` |
| macOS Intel | `apex-darwin-amd64.tar.gz` | `apex`, `apex-gui`, `apex-report` |
| Linux x64 | `apex-linux-amd64.tar.gz` | `apex` |
| Linux ARM64 | `apex-linux-arm64.tar.gz` | `apex` |
| Windows x86 | `apex-windows-386.zip` | `apex.exe` |
| Linux x86 | `apex-linux-386.tar.gz` | `apex` |

The desktop app is also published on its own, if you only want that binary:
`apex-gui-windows-amd64.exe`, `apex-gui-darwin-arm64`, `apex-gui-darwin-amd64`.

**There is no desktop build for Linux.** Wails needs GTK and WebKit at build
time and we do not ship that yet — on Linux, APEX is the CLI.

### Windows

Unzip and double-click `apex-gui.exe`. The desktop app renders through the
Microsoft WebView2 runtime, which ships with Windows 11 and with current
Windows 10; on an older machine, install
[WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)
first. `apex.exe` is the CLI and needs nothing.

### macOS

```bash
tar xzf apex-darwin-arm64.tar.gz
cd apex-darwin-arm64

# The binaries are not code-signed yet, so Gatekeeper quarantines them.
xattr -dr com.apple.quarantine .

./apex-gui          # desktop app
./apex --help       # CLI
```

Without the `xattr` line, macOS reports the app as damaged. Signing and
notarization are not in place yet.

### Linux

```bash
tar xzf apex-linux-amd64.tar.gz
cd apex-linux-amd64
./apex ./src --profile=essential
```

## Get started

### With the desktop app

1. **Scan** (`Ctrl/Cmd+1`) — pick the source folder with **Browse**, choose the
   profiles to run, set a minimum severity, and optionally point at a DDL file
   or folder to switch on cross-analysis. Press Run. Progress is shown and the
   scan can be cancelled.
2. **Results** (`Ctrl/Cmd+3`) — a dashboard of metrics and grades, plus the
   issue list. Clicking an issue opens the file at that line.
3. **Rules** (`Ctrl/Cmd+2`) — turn off rules that do not apply to this project,
   change a severity, or add a custom rule as a regex, testing it against a
   sample before saving. **Save the snapshot to a YAML file** — this is the
   file CI will use.
4. **Reports** (`Ctrl/Cmd+4`) — export Excel, HTML or JSON. Excel is the usual
   handover format; JSON is the input to the AI audit report.
5. **AI audit** (`Ctrl/Cmd+5`) — generate an audit report through an
   OpenAI-compatible endpoint, with air-gapped mode available. See
   [AI audit report](#ai-audit-report).

### From the command line

```bash
# Scan with every rule, straight to the console
apex ./src --profile=all

# Quality and security only, high severity and above
apex ./src --profile=quality,secure --min-severity=high

# Excel report — the usual handover format
apex ./src --profile=all -o excel --output-file=report.xlsx

# HTML and JSON
apex ./src --profile=all -o html --output-file=report.html
apex ./src --profile=all -o json --output-file=report.json

# Reuse the rule selection made in the desktop app
apex ./src --profile=all --overrides=ruleset.yaml -o excel --output-file=report.xlsx

# Tune on the command line instead, then save that selection for later
apex ./src --profile=all \
  --exclude-rule quality-nc-001,sql-fmt-003 \
  --min-severity=high \
  --save-overrides=ruleset.yaml

# DDL × query cross-analysis
apex ./src --profile=sql-ddl --ddl=./ddl/

# Korean output
apex ./src --profile=all --lang=ko
```

> The desktop app's interface is currently Korean only. The CLI follows
> `--lang` (`en` / `ko`) and defaults to your system locale.

## Profiles

Profiles group rules by purpose. Combine them with commas.

| Profile | What it covers | Rules | On by default |
|---|---|---|---|
| `quality` | Code quality — naming, logging, complexity, dead code | 125 | 107 |
| `secure` | Secure coding — injection, crypto, auth, error handling | 131 | 123 |
| `sql` | ANSI SQL, vendor independent | 120 | 102 |
| `sql-oracle` | Oracle specifics — NVL, hints, ROWNUM, DECODE | 20 | 18 |
| `sql-format` | SQL formatting and style | 7 | 7 |
| `modernize` | Legacy modernization — eGovFrame 3→4, javax→jakarta, iBatis→MyBatis | 50 | 39 |
| `spring` | Spring / Boot conventions | 32 | 28 |
| `egov` | eGovernment Framework standards | 33 | 12 |
| `ddl` | DDL × query cross-analysis (needs `--ddl`) | 17 | 17 |
| | **Total** | **535** | **453** |

Some rules ship switched off because they encode a team preference rather than
a defect — SQL formatting and several eGovFrame conventions, mostly. Turn them
on in the desktop app's Rules tab, or in an overrides file.

### Profile groups

| Group | Includes |
|---|---|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov |
| `essential` | quality, secure |
| `sql-all` | sql, sql-oracle, sql-format |
| `sql-ddl` | sql, sql-oracle, ddl |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring |
| `migration` | modernize, spring |

Run `apex profiles` to list them from the binary you have.

## What it checks

### Code quality (`quality`)

- Class naming — Controller, Service, ServiceImpl, Mapper, VO, Const, Util
- `System.out.println` prohibited; use a logger
- Logger string concatenation prohibited; use `{}` placeholders
- Empty catch blocks, dead code, god classes
- Method length, cyclomatic complexity
- MyBatis XML — `SQL_ID` comment required, `${}` prohibited, namespace required

### Secure coding (`secure`)

- SQL injection — `createStatement()` prohibited, use `PreparedStatement`
- XSS — `innerHTML`, `document.write`
- Command injection — `Runtime.exec()`, `ProcessBuilder` with external input
- Path traversal, SSRF, CSRF token verification
- Weak crypto — DES, MD5, SHA-1, RC4 prohibited; RSA 2048+, AES 128+
- Hardcoded passwords, keys and DB credentials
- `printStackTrace()` and `e.getMessage()` in responses

### SQL (`sql`, `sql-oracle`, `sql-format`)

- `SELECT *` prohibited, leading wildcard in `LIKE` prohibited
- Bind variables required; `UPDATE`/`DELETE` require a `WHERE`
- Code smells — `NATURAL JOIN`, `ON 1=1`, missing parentheses around `AND`/`OR`
- Oracle — hints, `NVL`/`TO_CHAR` index invalidation, `DECODE`, `ROWNUM` paging

### DDL × query cross-analysis (`ddl`)

Point `--ddl` at your DDL files and APEX cross-checks the schema against every
SQL statement it can find — MyBatis XML, standalone `.sql` files, and SQL
embedded in Java source (`@Select`/`@Insert`/`@Update`/`@Delete`, JPA native
queries, `JdbcTemplate`, raw JDBC, and plain string literals).

- **Query correctness** — tables and columns that do not exist, JOIN type
  mismatches causing implicit conversion, `INSERT` omitting a NOT NULL column,
  `UPDATE SET` on a primary key
- **Index usage** — WHERE/JOIN/GROUP BY/ORDER BY columns with no index,
  composite index leading-column skip, functions wrapping an indexed column
- **DDL improvements** — frequently queried columns with no index, FK without
  an index, unused or duplicate indexes, missing primary keys

```bash
apex ./src --profile=ddl --ddl=./ddl/schema.sql
apex ./src --profile=sql-ddl --ddl=./ddl/
```

In the desktop app this is the **DDL cross-analysis** field on the Scan tab.

## AI audit report

APEX turns a scan into a written audit report. It talks to any
OpenAI-compatible endpoint, so an in-house Ollama or vLLM server works and no
cloud API is involved.

**Desktop app** — run a scan, open the **AI audit** tab, enter the endpoint and
model, press **Test connection** to confirm the server answers, then generate.
Air-gapped mode is a checkbox.

**CLI** — two steps, the scan JSON first:

```bash
apex ./src --profile=all -o json --output-file=scan.json

apex ai-report scan.json --offline \
  --endpoint http://127.0.0.1:11434/v1 \
  --model qwen2.5-coder:7b \
  --project "Example System" \
  -o report.html
```

`--offline` installs a network guard before anything else runs. Connections to
anything but loopback are refused at the resolved IP, and DNS resolution is
blocked outright, so a hostname cannot slip past it. If the endpoint you gave
is external, the command fails and says so.

To confirm the guard is real rather than take our word for it, follow
[Air-gap verification](docs/AIRGAP_VERIFICATION.md) — it has you point APEX at
an external endpoint and watch it fail before running the local one.

| Flag | Meaning | Default |
|---|---|---|
| `-o`, `--output` | Report path | `ai-report.html` |
| `--provider` | `openai` (compatible), `claude`, `gemini` | `openai` |
| `--endpoint` | OpenAI-compatible endpoint | — |
| `--model` | Model name | — |
| `--api-key` | API key; any value works for a local server | — |
| `--project` | Name to use in the report | scan file name |
| `--lang` | `ko` / `en` | `ko` |
| `--offline` | Air-gapped mode; blocks non-loopback connections | `false` |
| `--timeout` | Per-request timeout | `5m` |

## CLI reference

```
apex [path] [flags]
apex profiles                       list profiles and groups
apex report [name:result.xlsx ...]  DOCX audit report from Excel results
apex ai-report [scan.json]          AI audit report from a scan JSON
```

| Flag | Short | Description | Default |
|---|---|---|---|
| `--profile` | `-p` | Profiles to run, comma-separated | `all` |
| `--profiles-file` | | Profile definition file | `configs/profiles.yaml` |
| `--config` | `-c` | Config file path | — |
| `--overrides` | | Apply a saved rule snapshot | — |
| `--save-overrides` | | Write the effective rule set to a YAML file | — |
| `--output` | `-o` | `console`, `json`, `html`, `excel` | `console` |
| `--output-file` | | Output path | stdout |
| `--min-severity` | `-s` | `low`, `medium`, `high`, `critical` | `low` |
| `--rules` | | Rule categories to check | — |
| `--exclude-rule` | | Rule IDs to exclude, comma-separated | — |
| `--ddl` | | DDL file or directory, enables cross-analysis | — |
| `--cross-file-only` | | Cross-file analysis only | `false` |
| `--summary` | | Per-rule summary table, useful in CI | `false` |
| `--no-dedup` | | Keep duplicate findings | `false` |
| `--lang` | | Output language, `en` / `ko` | system locale |
| `--verbose` | `-v` | Verbose output | `false` |

## Desktop app reference

| Tab | Shortcut | What it does |
|---|---|---|
| Scan | `Ctrl/Cmd+1` | Source folder, profiles, minimum severity, DDL cross-analysis; run, cancel, progress |
| Rules | `Ctrl/Cmd+2` | Enable/disable rules, change severity, add custom regex rules with a live tester, save and load snapshots |
| Results | `Ctrl/Cmd+3` | Metrics dashboard with grades, issue list, open a file at the offending line |
| Reports | `Ctrl/Cmd+4` | Export Excel / HTML / JSON, recent exports |
| AI audit | `Ctrl/Cmd+5` | Endpoint and model, connection test, air-gapped mode, progress, cancel |
| Settings | `Ctrl/Cmd+6` | Application preferences |

## GitHub Action

```yaml
- uses: mhb8436/apex-ai@v1
  with:
    path: './src'
    profile: 'essential'
    min-severity: 'medium'
    lang: 'en'
```

Pair it with a snapshot from the desktop app to run exactly the rules your team
agreed on:

```yaml
- run: apex ./src --profile=all --overrides=ruleset.yaml --summary --min-severity=high
```

## Documentation

- [Quick start](docs/QUICK_START.md) — desktop app and CLI, from install to first report
- [User manual](docs/APEX_사용자_매뉴얼.md) — every screen, every flag, every rule category
- [Custom rules](docs/CUSTOM_RULES.md) — writing your own rules
- [Air-gap verification](docs/AIRGAP_VERIFICATION.md) — proving the offline guard works

## License

Dual-licensed:

- **AGPL-3.0** — free for open-source use
- **Commercial license** — for proprietary use

APEX is built by CRAFTICSYSTEMS Co., Ltd. For commercial licensing or support,
open an issue on this repository.
