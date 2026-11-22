# pdf2ko

pdf2ko는 Codex로 영문 PDF를 한국어로 번역하는 프로젝트입니다.
번역을 요청하면 Codex가 문서별 폴더(`documents/<id>/`)에 용어집과 본문 대조표를 작성합니다. 두 파일을 검토하고 저장한 뒤 한글 PDF 생성을 요청하면, 본문 대조표에 저장한 번역을 원본 PDF에 반영합니다.

| 파일 | 용도 |
| --- | --- |
| `review/glossary.md` | **용어집**. 영어 용어와 한국어 표기를 짝지어 정리합니다. 같은 용어를 문서 전체에서 어떻게 옮길지 정할 때 봅니다. |
| `review/trans.md` | **본문 대조표**. PDF 페이지별로 영어 원문과 한국어 번역을 나란히 보여 줍니다. 한글 PDF에 들어갈 문장을 여기서 다듬습니다. |


## 번역 예시

**NumPy로 하는 배열 프로그래밍**은 pdf2ko로 영문 논문을 한국어로 번역하고 지면에 맞춰 다시 조판한 비공식 번역 예시입니다.

[영문 원본 PDF](examples/Array_programming_with_NumPy.pdf) · [한국어 번역 PDF (v001)](examples/array-programming-with-numpy-ko-v001.pdf)

원문은 Charles R. Harris 외 저자들의 「Array programming with NumPy」로, *Nature* 585, 357–362 (2020)에 실렸습니다. © The Author(s) 2020. [원문 DOI](https://doi.org/10.1038/s41586-020-2649-2) · [원문 라이선스 CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)


## 번역 요청부터 한글 PDF까지

1. **번역을 요청합니다.** `inbox/`에 영문 PDF를 넣고 이 프로젝트를 연 Codex에 “`inbox/Example Rules.pdf`를 번역해 줘.”라고 요청합니다.
2. **두 파일을 검토하고 저장합니다.** 용어집과 본문 대조표에서 `한국어 번역` 열을 수정합니다. 용어집에서 ‘게임판’을 ‘보드’로 바꿨다면 본문 대조표에서도 해당 표현을 바꿉니다.
3. **한글 PDF 생성을 요청합니다.** “저장한 번역으로 `Example Rules.pdf`의 한글 PDF를 만들어 줘.”라고 요청하면 Codex가 현재 저장된 번역을 원본에 반영합니다.
4. **결과를 확인합니다.** 전달받은 PDF를 열어 번역 누락, 글자 잘림, 표나 그림과의 겹침을 확인합니다. 결과는 `documents/<id>/output/<id>-ko-v001.pdf`처럼 버전별로 저장됩니다.


## 실행 환경 준비

준비할 항목은 **Python 의존성 설치 → PdfCraft CLI 설치**입니다. 번역 검토 파일 작성에는 Python 환경이 필요하고, 한글 PDF 편집에는 PdfCraft CLI를 직접 실행합니다.

`requirements.txt`는 Python 패키지만 설치합니다. 여기서 사용하는 PdfCraft는 [storytold/pdfcraft](https://github.com/storytold/pdfcraft)의 Rust 프로그램이며 실행 파일 이름은 **`pdfcraft-cli`**입니다. `pip install pdfcraft`나 이름이 비슷한 패키지로 대체하지 않습니다.

### 1. Python 환경 준비

저장소 최상위 폴더에서 실행합니다. macOS/Linux의 `sh` 호환 셸 예시입니다.

```sh
export PDF2KO_PYTHON=python3
python3 -m pip install -r requirements.txt
scripts/setup.sh
```

Windows에서는 Python과 Git Bash를 준비하고, Git Bash에서 위 명령의 `python3`를 `python`으로 바꿔 실행합니다. 이후 프로젝트의 `scripts/*.sh` 명령도 Git Bash에서 실행합니다.

새 터미널에서도 `export PDF2KO_PYTHON=python3`을 설정합니다(Windows는 `python`). 이 변수는 프로젝트 스크립트가 사용할 Python을 지정합니다. `setup.sh`는 `pypdf`, `pdfplumber`, `reportlab`을 가져올 수 있는지 확인하고 `inbox/`와 `documents/`를 준비합니다. **PdfCraft CLI는 아래 안내에 따라 별도로 설치합니다.**

### 2. PdfCraft CLI 설치

아래 방법 중 하나를 선택합니다. 설치 안내 확인일은 **2026-11-22**이며, 당시 [공식 최신 릴리스는 v0.21.1](https://github.com/storytold/pdfcraft/releases/tag/v0.21.1)입니다. 아래 버전은 설치 예시이며 pdf2ko의 최소 지원 버전을 뜻하지 않습니다. 다른 버전을 설치할 때는 [Releases의 Assets](https://github.com/storytold/pdfcraft/releases/latest)에서 실제 파일 이름을 확인합니다.

#### macOS: Homebrew

Homebrew가 설치되어 있다면 [공식 Formula의 설치 명령](https://formulae.brew.sh/formula/pdfcraft-cli)을 사용합니다.

```sh
brew install pdfcraft-cli
command -v pdfcraft-cli
pdfcraft-cli --version
pdfcraft-cli --help
```

이미 Homebrew로 설치했다면 `brew upgrade pdfcraft-cli`로 업데이트합니다. Homebrew의 지원 OS와 CPU, 바이너리 제공 여부는 Formula 페이지에서 확인합니다. 지원되지 않는 환경에서는 아래 공식 배포 파일을 사용합니다.

#### macOS/Linux: 공식 CLI 압축 파일

v0.21.1의 실제 배포 파일은 다음과 같습니다. macOS의 DMG는 데스크톱 앱용이고 CLI는 별도 ZIP으로 배포됩니다. Linux에서 CLI만 사용할 때는 전용 tar.gz를 선택할 수 있습니다. [공식 다운로드 안내](https://github.com/storytold/pdfcraft#downloads)

| 환경 | CLI 배포 파일 |
| --- | --- |
| macOS, Apple silicon 또는 Intel | `pdfcraft-cli-0.21.1-macos-universal.zip` |
| Linux, x86_64 | `pdfcraft-cli-0.21.1-linux-x86_64.tar.gz` |
| Linux, ARM64 / aarch64 | `pdfcraft-cli-0.21.1-linux-aarch64.tar.gz` |

Linux x86_64/aarch64 배포 파일은 glibc 2.35 이상이 필요합니다. CPU는 `uname -m`, glibc 버전은 glibc 기반 배포판에서 `ldd --version`으로 확인할 수 있습니다.

1. 선택한 파일과 같은 릴리스의 `SHA256SUMS.txt`를 내려받습니다. macOS는 `shasum -a 256 파일명`, Linux는 `sha256sum 파일명`으로 해시를 구해 해당 파일의 기록과 비교합니다.
2. ZIP은 압축 해제 도구 또는 `unzip 파일명.zip`, tar.gz는 `tar -xzf 파일명.tar.gz`로 풉니다.
3. 풀린 폴더에서 `pdfcraft-cli`를 찾고 아래 첫 번째 경로를 실제 파일의 절대 경로로 바꿔 설치합니다.

```sh
mkdir -p "$HOME/.local/bin"
install -m 755 "/absolute/path/to/unpacked/pdfcraft-cli" "$HOME/.local/bin/pdfcraft-cli"
export PATH="$HOME/.local/bin:$PATH"
command -v pdfcraft-cli
pdfcraft-cli --version
pdfcraft-cli --help
```

`export PATH=...`는 현재 셸에만 적용됩니다. 계속 사용할 셸의 시작 설정에도 추가하거나, 명령을 실행할 때 실행 파일의 절대 경로를 사용합니다. 수동 설치를 업데이트할 때도 새 배포 파일을 받아 같은 설치 위치를 사용합니다.

Linux에서 `.deb` 또는 `.rpm`을 사용하면 `/usr/bin/pdfcraft-cli`가 함께 설치됩니다. 예를 들어 v0.21.1 x86_64 패키지를 다운로드한 폴더에서 Debian/Ubuntu는 `sudo apt install ./pdfcraft-0.21.1-linux-x86_64.deb`, Fedora는 `sudo dnf install ./pdfcraft-0.21.1-linux-x86_64.rpm`으로 설치합니다. ARM64는 파일명의 `x86_64`를 `aarch64`로 바꿉니다.

#### Windows: MSI 또는 portable ZIP

[v0.21.1 릴리스](https://github.com/storytold/pdfcraft/releases/tag/v0.21.1)에서 CPU에 맞는 파일을 받습니다. 이 릴리스에서 확인되는 Windows 아키텍처는 `x64`와 `arm64`입니다.

| 방식 | 파일명 (`<arch>`는 `x64` 또는 `arm64`) |
| --- | --- |
| 설치 프로그램 | `pdfcraft-0.21.1-windows-<arch>.msi` |
| 압축 해제 후 사용 | `pdfcraft-0.21.1-windows-<arch>-portable.zip` |

MSI 또는 portable ZIP에는 `pdfcraft-cli.exe`가 포함됩니다. MSI는 설치할 때 선택한 폴더, ZIP은 압축을 푼 폴더에서 실행 파일을 찾습니다. ZIP은 폴더 전체를 보관합니다. [공식 Windows 패키징 코드](https://github.com/storytold/pdfcraft/blob/v0.21.1/packaging/windows/package.ps1)

실행 파일이 들어 있는 폴더를 사용자 `Path`에 추가하고 새 PowerShell에서 확인합니다.

```powershell
Get-Command pdfcraft-cli
pdfcraft-cli --version
pdfcraft-cli --help
```

`Path`에 추가하지 않을 때는 PowerShell에서 `& 'C:\실제 설치 폴더\pdfcraft-cli.exe' --version`처럼 전체 경로로 실행합니다. 다운로드 해시는 `Get-FileHash .\다운로드한파일 -Algorithm SHA256`으로 구해 해당 릴리스의 `SHA256SUMS.txt`와 비교할 수 있습니다.

#### 소스에서 설치해야 하는 경우

저장소를 받은 뒤 다음 명령으로 CLI를 설치합니다. 먼저 Git, Rust/Cargo와 OS의 네이티브 링커가 필요합니다. 확인일 기준 upstream이 안내하는 Rust 최소 버전은 1.95이며, 체크아웃한 버전의 [개발 문서](https://github.com/storytold/pdfcraft/blob/main/docs/development.md)도 확인합니다.

```sh
# pdf2ko 폴더 밖에서 PdfCraft 소스를 내려받습니다.
git clone https://github.com/storytold/pdfcraft.git
cd pdfcraft
cargo install --path apps/pdfcraft-cli --locked
export PATH="${CARGO_HOME:-$HOME/.cargo}/bin:$PATH"
pdfcraft-cli --version
pdfcraft-cli --help
```

기본 설치 위치는 macOS/Linux의 `~/.cargo/bin`, Windows의 `%USERPROFILE%\.cargo\bin`이며 Cargo 설치 경로를 변경했다면 그 경로를 사용합니다. `target/release`를 가리키는 링크는 `cargo clean`으로 깨질 수 있으므로 설치된 실행 파일을 사용합니다.

### 3. PdfCraft CLI 실행

Codex는 터미널에서 `pdfcraft-cli`를 직접 실행합니다. 사용 가능한 명령과 편집 인자는 다음 명령으로 확인합니다.

```sh
pdfcraft-cli --help
pdfcraft-cli tools
```

여러 편집 작업은 `pdfcraft-cli run --script steps.json`으로 실행할 수 있습니다. 실행 스크립트는 `tool`과 `args`를 가진 단계 배열이며, `doc_open`으로 열기부터 편집과 `doc_save`로 저장까지 한 실행에 묶습니다. 저장 경로에는 `prepare-edit`가 반환한 `candidate_pdf`의 절대 경로를 사용합니다. 실제 편집에서는 설치된 버전의 명령·인자와 지원 기능을 확인한 뒤 프로젝트의 [편집 절차](agents/editor.md)를 따릅니다.

### 4. 설치·실행 문제 해결

| 증상 | 확인할 사항 |
| --- | --- |
| Python 의존성 설치 후에도 `pdfcraft-cli`를 찾지 못함 | Python 패키지와 별도로 위 CLI 설치를 수행했는지 확인합니다. |
| PdfCraft 앱은 열리지만 CLI 명령을 찾지 못함 | 앱 실행 파일 `pdfcraft`와 CLI `pdfcraft-cli`를 구분하고, 해당 OS의 CLI 설치 위치를 확인합니다. |
| 터미널에서는 되지만 Codex에서 실행 파일을 찾을 수 없다는 오류 | Codex가 상속한 `PATH`가 다를 수 있습니다. CLI의 절대 경로로 실행하거나 `PATH`를 수정한 뒤 Codex를 다시 시작합니다. |
| 예상과 다른 버전이 실행됨 | macOS/Linux는 `type -a pdfcraft-cli`, PowerShell은 `Get-Command pdfcraft-cli -All`로 중복 설치를 확인하고 실제 실행 경로를 비교합니다. |
| CLI를 실행할 수 없거나 필요한 편집 기능을 지원하지 않음 | 설치 버전과 `pdfcraft-cli --help`, `pdfcraft-cli tools`를 확인합니다. PDF 편집은 중단하고 원인을 알리며 다른 엔진으로 우회하지 않습니다. |

## 검토 파일 편집 예시

두 파일 모두 아래와 같은 표 하나로 구성됩니다. 표 머리글, ID, PDF 페이지, 영어 원문은 그대로 두고 한국어 번역을 수정합니다.

**용어집** (`review/glossary.md`)

| 용어 id | 영어 용어 | 한국어 번역 |
| --- | --- | --- |
| term-001 | board | 게임판 |

**본문 대조표** (`review/trans.md`)

| 문단id | pdf페이지 | 영어 원문 | 한국어 번역 |
| --- | --- | --- | --- |
| p001-s001 | 1 | Place the board on the table. | 게임판을 탁자 위에 놓습니다. |

본문의 한 행은 문단, 제목, 목록 항목, 그림 설명처럼 의미가 통하는 한 단위입니다. 페이지 번호는 PDF의 첫 페이지를 1로 셉니다.

셀 안에서 줄을 바꿀 때는 `<br>`를 씁니다. 세로줄과 역슬래시는 각각 `\|`와 `\\`로, 문자 그대로의 `&`, `<`, `>`는 각각 `&amp;`, `&lt;`, `&gt;`로 씁니다.

## 명령어 참고

번역과 PDF 편집은 Codex가 수행합니다. 아래는 그 과정에서 쓰는 문서 관리 명령입니다. 예시 문서 ID는 `example-rules`입니다.

```sh
# 문서를 등록하고 원문 텍스트와 위치 확인
scripts/run.sh register --pdf "inbox/Example Rules.pdf" --id example-rules --kind boardgame
scripts/run.sh extract example-rules

# 검토 파일을 저장한 뒤 표 구조와 문서 상태 확인
scripts/run.sh validate example-rules
scripts/run.sh status example-rules

# PDF 생성 요청 후 편집 버전 준비
scripts/run.sh prepare-edit example-rules

# PdfCraft로 편집본을 저장한 뒤 결과 폴더에 복사 (첫 버전 예시)
scripts/run.sh publish example-rules --version v001 \
  --pdf documents/example-rules/work/edit-v001/candidate.pdf
```

- `register`는 새 원본을 문서의 `source/`로 옮깁니다. 내용이 같은 PDF가 이미 등록되어 있으면 기존 문서를 사용합니다. 문서 종류(`--kind`)는 `boardgame`, `"research paper"`, `other` 중에서 선택합니다.
- `extract`는 추출 결과를 표준 출력으로 보여 줍니다. 텍스트가 없는 스캔 페이지는 번역할 때 OCR과 원본 대조가 필요합니다.
- `prepare-edit`는 원본과 검토 파일의 해시를 기록합니다. 진행 중인 버전은 `--version v001`처럼 지정해 재개합니다. 입력이나 편집 작업 형식이 바뀌었다면 `--version` 없이 새 버전을 준비합니다.
- 전체 문서 상태는 `scripts/run.sh status`로 확인합니다. 다른 프로젝트 폴더를 사용하려면 `scripts/run.sh --root /path/to/project status`처럼 명령 이름 앞에 `--root`를 넣습니다.

에이전트의 작업 절차와 파일 보존 규칙은 [AGENTS.md](AGENTS.md)에 있습니다. 구현을 변경한 뒤에는 `scripts/test.sh`로 테스트를 실행할 수 있습니다.
