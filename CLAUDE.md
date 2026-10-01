# 분당우리교회 출석 데이터 입력앱

교역자가 매주 출석·헌금을 입력하는 화면. HTML 파일 **하나**(`index.html`, 약 6,000줄)로
되어 있고 빌드 과정이 없다. 저장한 그대로 올라간다.

- 운영 주소 — https://woorichurchplanning-dev.github.io/bwc-input/
- 배포 — `main` 에 push 하면 GitHub Pages 가 1~2분 안에 반영한다. 그게 전부다.
- 자료가 들어가는 곳 — 구글시트 `bwc_data_input` (Apps Script 경유)

## 함께 봐야 할 두 곳

이 앱만 고쳐서 끝나는 일이 드물다. 자료가 지나는 길이 셋으로 나뉘어 있다.

| | 무엇 | 어디 |
|---|---|---|
| 입력앱 | 이 레포 | GitHub Pages |
| Apps Script | 시트를 읽고 쓰는 중간층 | `bwc-gas` 레포 · clasp |
| 대시보드 | 보는 화면 | `bwc-dashboard` 레포 · Vercel |

**입력 항목을 더하거나 이름을 바꾸면 세 곳을 같이 고쳐야 한다.** 입력앱의 `DEPARTMENTS`
에만 넣으면 시트에는 들어가지만 대시보드에 안 나온다.

## 꼭 알아야 할 것

**주차(weekKey) 규칙** — 그 한 주가 **끝나는 주일** 날짜다. 9/21(월)~9/27(일) → `2026-09-27`.
수요예배는 주일 **이전** 수요일(9/23), 새벽기도는 월~토(9/21~9/26)다. `sundayOfWeek()` 가 기준.
날짜를 주차로 바꾸는 코드를 새로 쓰지 말고 이 함수를 쓸 것.

**값은 처음에 안 받아 온다** — 들어올 때는 '누가 냈는지'만 받는다(`mode=status`).
숫자는 폼이나 표를 열 때 그때 받는다(`ensureItemValues` / `ensureFormHistory` /
`ensureGroupValues`). 화면에 숫자를 새로 보여 줄 일이 생기면 **이 중 하나를 반드시 불러야
한다.** 안 부르면 시트에 있어도 빈 칸으로 보인다 — 이 앱에서 가장 자주 났던 고장이다.

**전체 조회 키는 관리자 비밀번호로 잠겨 있다** (`unlockReadKey`). 평소 입력에는 키가 필요
없다. 「데이터 현황」 탭에 로그인해야 풀린다. 아이디는 `admin`, 비밀번호는 따로 전달받을 것
— 소스에는 해시(`ADMIN_PW_HASH`)만 있다. 바꾸려면 콘솔에서 `genAdminHash('새비밀번호')`.

**이 레포는 공개(public)다.** 교역자 이름·연락처·비밀번호·출석 숫자를 커밋하지 말 것.

## 고칠 때

- 화면은 `switchTab()` 으로 나뉜다: `input`(교구/청년교구) · `sunday-school` · `special` ·
  `status`(제출 현황) · `admin`(데이터 현황)
- 일반 폼은 `openForm()`, 표 화면은 `openWorshipGrid` / `openAgeGrid` /
  `openDistrictGrid` / `openSpecialForm` 으로 따로 간다
- 제출은 `submitForm()` → Apps Script `doPost`. 숫자는 `formData`, 비고는 `notes` 로 나간다
- 확인은 Playwright 로. **헤드리스 기본 UA 는 막혀 있으니** 평범한 크롬 UA 를 쓸 것
- 시트를 건드리지 않고 확인하려면 POST 를 가로채 끊는다:
  `page.route('**/script.google.com/macros/**', r => r.request().method()==='POST' ? r.abort() : r.continue())`
