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


## 실행 환경 준비

PDF 편집에 사용하는 [PdfCraft CLI](https://github.com/storytold/pdfcraft)를 설치합니다.

### macOS

Homebrew로 설치합니다.

```sh
brew install pdfcraft-cli
```

### Windows

1. [공식 릴리스의 Assets](https://github.com/storytold/pdfcraft/releases/latest)에서 CPU에 맞는 Windows용 `x64` 또는 `arm64` MSI/portable ZIP을 받습니다.
2. MSI를 실행하거나 ZIP 전체를 압축 해제합니다.
3. `pdfcraft-cli.exe`가 있는 폴더를 사용자 환경 변수 `Path`에 추가한 뒤 터미널과 Codex를 다시 엽니다.

### 설치 후 공통 확인

```sh
pdfcraft-cli --version
pdfcraft-cli --help
pdfcraft-cli tools
```
