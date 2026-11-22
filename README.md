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
- `publish`는 입력 해시·페이지 수·복사 무결성과 페이지별 이미지 전체 소실 여부를 검사합니다. 원본 페이지 콘텐츠에 있던 이미지 그리기 명령이 후보의 같은 페이지에서 전부 사라지면 게시를 차단합니다. 중첩 Form과 인라인 이미지도 확인하며, 사용되지 않는 이미지 리소스는 세지 않습니다. 이미지 개수 일치로 보존을 판정하는 검사가 아니며, 일부 손실·가림·잘림·화면 밖 배치는 최종 시각 검수에서 사람이 확인합니다.
- 전체 문서 상태는 `scripts/run.sh status`로 확인합니다. 다른 프로젝트 폴더를 사용하려면 `scripts/run.sh --root /path/to/project status`처럼 명령 이름 앞에 `--root`를 넣습니다.

에이전트의 작업 절차와 파일 보존 규칙은 [AGENTS.md](AGENTS.md)에 있습니다. 구현을 변경한 뒤에는 `scripts/test.sh`로 테스트를 실행할 수 있습니다.


## 실행 환경 준비

macOS와 Windows에서 **Python 의존성 설치 → PdfCraft CLI 설치** 순서로 준비합니다. `requirements.txt`는 Python 패키지만 설치합니다. PDF 편집과 테스트에 사용하는 **`pdfcraft-cli`**는 [storytold/pdfcraft](https://github.com/storytold/pdfcraft)의 별도 실행 파일입니다.

### macOS

#### 1. Python 환경 준비

Python 3를 설치하고 저장소 최상위 폴더의 터미널에서 실행합니다. 패키지는 프로젝트의 `.venv` 가상 환경에 설치합니다.

```sh
python3 -m venv .venv
export PDF2KO_PYTHON="$PWD/.venv/bin/python3"
"$PDF2KO_PYTHON" -m pip install -r requirements.txt
scripts/setup.sh
```

새 터미널에서도 저장소로 이동한 뒤 위 `export` 명령을 실행합니다. `.venv` 생성과 패키지 설치를 매번 반복할 필요는 없습니다.

#### 2. PdfCraft CLI 설치

Homebrew가 있다면 [공식 Formula](https://formulae.brew.sh/formula/pdfcraft-cli)의 명령으로 설치합니다.

```sh
brew install pdfcraft-cli
command -v pdfcraft-cli
pdfcraft-cli --version
```

이미 설치했다면 `brew upgrade pdfcraft-cli`로 업데이트합니다.

수동 설치는 [공식 릴리스](https://github.com/storytold/pdfcraft/releases/latest)의 macOS CLI ZIP을 사용합니다. v0.21.1의 파일명은 `pdfcraft-cli-0.21.1-macos-universal.zip`이며 Apple silicon과 Intel용입니다. DMG는 데스크톱 앱용입니다.

1. CLI ZIP을 내려받아 압축을 풉니다.
2. 아래 첫 번째 경로를 압축을 푼 `pdfcraft-cli`의 실제 경로로 바꿔 실행합니다.

```sh
mkdir -p "$HOME/.local/bin"
install -m 755 "/absolute/path/to/unpacked/pdfcraft-cli" "$HOME/.local/bin/pdfcraft-cli"
export PATH="$HOME/.local/bin:$PATH"
command -v pdfcraft-cli
pdfcraft-cli --version
```

수동 설치의 `PATH` 설정을 유지하려면 사용하는 셸의 시작 파일에 같은 `export PATH` 줄을 추가합니다. 기본 zsh에서는 `~/.zshrc`를 사용합니다.

### Windows

#### 1. Python과 Git Bash 준비

1. [Python 3](https://www.python.org/downloads/windows/)와 [Git for Windows](https://gitforwindows.org/)를 설치합니다. Git for Windows에 포함된 **Git Bash**에서 프로젝트의 `scripts/*.sh` 명령을 실행합니다.
2. Git Bash를 새로 열고 저장소 폴더로 이동합니다. 아래 경로는 예시이므로 실제 위치로 바꿉니다. Git Bash에서 `C:\Users\사용자\Documents\pdf2ko`는 `/c/Users/사용자/Documents/pdf2ko`로 쓸 수 있습니다. 공백이 있는 경로는 따옴표로 감쌉니다.
3. `python --version`이 Python 3 버전을 출력하는지 확인합니다.

```sh
cd "/c/Users/사용자/Documents/pdf2ko"
python --version
```

`python`을 찾지 못하거나 Microsoft Store가 열리면 Python 설치와 `PATH`를 확인하고 Git Bash를 다시 엽니다. `py --version`만 정상 동작한다면 다음 단계의 첫 명령을 `py -3 -m venv .venv`로 바꿔 실행할 수 있습니다.

#### 2. 프로젝트 전용 Python 환경 만들기

**Git Bash에서** 실행합니다. 아래의 `export`와 `.sh` 명령은 PowerShell이나 명령 프롬프트(cmd)에 그대로 입력하지 않습니다.

```sh
python -m venv .venv
export PDF2KO_PYTHON="$PWD/.venv/Scripts/python.exe"
"$PDF2KO_PYTHON" -m pip install -r requirements.txt
scripts/setup.sh
```

Windows 가상 환경의 Python은 `.venv/Scripts/python.exe`에 있습니다. `PDF2KO_PYTHON`으로 이 실행 파일을 지정하므로 별도 활성화 명령 없이 패키지 설치와 프로젝트 실행에 같은 Python을 사용합니다. [Python 가상 환경 문서](https://docs.python.org/3/library/venv.html)

`setup.sh`는 `pypdf`, `pdfplumber`, `reportlab`을 가져올 수 있는지 확인하고 `inbox/`, `documents/`를 준비합니다. PdfCraft CLI는 다음 단계에서 설치합니다.

새 Git Bash 창을 열면 저장소로 이동한 뒤 아래 설정을 다시 실행합니다. 가상 환경 생성과 패키지 설치는 반복하지 않아도 됩니다.

```sh
export PDF2KO_PYTHON="$PWD/.venv/Scripts/python.exe"
scripts/run.sh status
```

#### 3. PdfCraft CLI 설치

[공식 릴리스의 Assets](https://github.com/storytold/pdfcraft/releases/latest)에서 CPU에 맞는 **MSI 또는 portable ZIP 중 하나**를 받습니다. Windows의 **설정 → 시스템 → 정보 → 시스템 종류**에서 CPU 종류를 확인합니다. x64 기반 PC는 `x64`, ARM 기반 PC는 `arm64`를 선택합니다.

아래는 [v0.21.1](https://github.com/storytold/pdfcraft/releases/tag/v0.21.1)의 파일명 예시입니다. 다른 버전은 해당 릴리스의 실제 파일명을 확인합니다.

| 방식 | x64 | ARM64 |
| --- | --- | --- |
| 설치 프로그램 | `pdfcraft-0.21.1-windows-x64.msi` | `pdfcraft-0.21.1-windows-arm64.msi` |
| 압축 해제 후 사용 | `pdfcraft-0.21.1-windows-x64-portable.zip` | `pdfcraft-0.21.1-windows-arm64-portable.zip` |

- **MSI:** 설치 프로그램을 실행한 뒤 설치 폴더에서 `pdfcraft-cli.exe`를 찾습니다.
- **Portable ZIP:** 원하는 폴더에 **전체 압축을 해제**하고 그 안에서 `pdfcraft-cli.exe`를 찾습니다. 실행 파일만 따로 옮기지 말고 압축을 푼 폴더 전체를 보관합니다.

두 배포 형식에는 CLI가 포함됩니다. [공식 Windows 패키징 코드](https://github.com/storytold/pdfcraft/blob/v0.21.1/packaging/windows/package.ps1)

#### 4. Path 등록과 실행 확인

1. 시작 메뉴에서 **환경 변수**를 검색하고 **계정의 환경 변수 편집**을 엽니다.
2. **사용자 변수 → Path → 편집 → 새로 만들기**에서 `pdfcraft-cli.exe`가 들어 있는 폴더의 실제 경로를 추가합니다. 예를 들어 파일이 `C:\Tools\PdfCraft\pdfcraft-cli.exe`에 있다면 `C:\Tools\PdfCraft`를 추가합니다.
3. 설정을 저장한 뒤 Git Bash와 Codex를 다시 엽니다.
4. 새 Git Bash에서 확인합니다.

```sh
command -v pdfcraft-cli
pdfcraft-cli --version
pdfcraft-cli --help
```

PowerShell에서 설치 위치를 확인할 때는 `Get-Command pdfcraft-cli`를 사용합니다. 프로젝트의 `.sh` 명령은 계속 Git Bash에서 실행합니다.

### 설치 후 공통 확인

저장소에서 운영체제에 맞게 `PDF2KO_PYTHON`을 설정한 뒤 실행합니다.

```sh
pdfcraft-cli --help
pdfcraft-cli tools
scripts/test.sh
```

테스트는 임시 폴더에서 입력·출력 무결성을 확인합니다. 실제 문서의 번역·시각 검수를 대신하지 않습니다.

여러 PDF 편집 작업은 `pdfcraft-cli run --script steps.json`으로 실행합니다. 스크립트는 `tool`과 `args`를 가진 단계 배열이며, `doc_open`으로 열기부터 편집과 `doc_save`로 저장까지 한 실행에 묶습니다. 저장 경로에는 `prepare-edit`가 반환한 `candidate_pdf`의 절대 경로를 사용합니다. 실제 편집은 [편집 절차](agents/editor.md)를 따릅니다.

### 설치·실행 문제 해결

| 증상 | 확인할 사항 |
| --- | --- |
| `scripts/setup.sh`에서 Python을 찾지 못함 | 저장소 최상위 폴더에서 해당 OS의 `export PDF2KO_PYTHON=...` 명령을 실행했는지 확인합니다. |
| Python 패키지를 설치했는데 `pdfcraft-cli`가 없음 | CLI는 별도로 설치합니다. `pip install pdfcraft` 등 이름이 비슷한 패키지로 대체하지 않습니다. |
| Windows에서 `export` 또는 `.sh`를 실행할 수 없음 | PowerShell/cmd 대신 Git Bash에서 실행합니다. |
| PdfCraft 앱은 열리지만 CLI를 찾지 못함 | `pdfcraft-cli.exe`가 있는 폴더를 사용자 `Path`에 추가하고 Git Bash와 Codex를 다시 엽니다. macOS에서는 `command -v pdfcraft-cli`로 확인합니다. |
| 예상과 다른 CLI 버전이 실행됨 | macOS/Git Bash는 `type -a pdfcraft-cli`, PowerShell은 `Get-Command pdfcraft-cli -All`로 설치 경로를 확인합니다. |
| 필요한 PDF 편집 기능을 지원하지 않음 | `pdfcraft-cli --help`와 `pdfcraft-cli tools`에서 지원 기능을 확인합니다. 필요한 기능이 없으면 편집을 중단하고 원인을 알립니다. |
