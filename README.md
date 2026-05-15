# 콘텐츠팀 운영 OS

분기 베팅(BET) 중심의 운영 OS. BET → 대 카테고리 → 소 카테고리 3단 위계로 진행 상황을 관리하고, 페이즈 정체와 졸업 신호를 시각적으로 추적합니다.

---

## 🗂 파일 구조

```
content-team-os/
├── index.html                          # UI + 로직 (단일 파일)
├── data.json                           # 베팅·페이즈·카테고리 데이터
├── README.md                           # 본 문서
└── .github/
    └── workflows/
        └── stagnation-alert.yml        # 슬랙 알림 (매일 새벽)
```

**핵심 원칙**: `index.html`은 거의 안 건드리고 `data.json`만 자주 수정한다.
UI 로직과 데이터를 분리해서 *모바일에서도 데이터만 빠르게 수정* 할 수 있게 설계했다.

---

## 🚀 셋업 (10분)

### 1. GitHub 리포 만들기

1. GitHub에서 새 리포 생성 (Public 또는 Private 둘 다 가능, Pages는 Public이 무료)
2. 이 폴더의 모든 파일을 리포에 업로드 (또는 git clone 후 푸시)

### 2. GitHub Pages 활성화

1. 리포의 **Settings** → **Pages**
2. Source를 **Deploy from a branch** → `main` / `(root)`
3. 1~2분 뒤 `https://<유저명>.github.io/<리포명>/` 에서 라이브 접속 가능

### 3. 슬랙 알림 활성화 (선택)

아래 [슬랙 알림](#-슬랙-알림-셋업) 섹션 참고.

---

## 📱 모바일/아이패드에서 수정하기

**가장 권장: `github.dev` 사용** (VSCode 환경이 브라우저에서 그대로 열림. 무료. 추가 앱 설치 X)

### 방법

1. 깃허브에서 본 리포 페이지 열기 (Safari/Chrome 모두 가능)
2. URL에서 `github.com` 을 `github.dev` 로 바꾸기
   - 예: `https://github.com/유저명/리포명` → `https://github.dev/유저명/리포명`
   - 또는 리포 화면에서 키보드 `.` 키 한 번
3. 브라우저에서 VSCode 환경이 열림
4. `data.json` 열어서 수정
5. 좌측 Source Control 아이콘 → 메시지 입력 → ✓ Commit & Push
6. 1~2분 뒤 라이브 사이트에 반영됨

### 자주 수정하는 곳

| 무엇을 바꾸려면 | 어디를 열어서 | 어디를 수정 |
|---|---|---|
| 매출/조회수/팔로워 숫자 | `data.json` | `metrics.revenue.value` 등 |
| 이번 주 안건 | `data.json` | `metrics.thisWeek` |
| BET 타이틀/목표 | `data.json` | `bets[].title`, `bets[].goal` |
| 페이즈 상태 (사이트에서 클릭으로도 가능) | `data.json` | `bets[].phases[].status` |
| 카테고리/소 카테고리 레벨 추가 | `data.json` | `bets[].categories[].levels[]` 또는 `subCategories[].levels[]` |
| 정체 임계일 변경 | `data.json` | `bets[].phases[].threshold` |
| Sustain 트랙 추가 | `data.json` | `sustainTracks[]` |

### 사이트에서 클릭으로 수정한 경우

- 클릭 변경은 **브라우저 localStorage**에만 저장됨 (해당 디바이스에만)
- 영구 반영하려면: 상단 **⬇ data.json 내려받기** 버튼 → 받은 파일을 깃에 푸시
- 또는 다른 디바이스에서 그대로 직접 `data.json`을 편집

---

## 🔔 슬랙 알림 셋업

### 1. Slack에서 Incoming Webhook 만들기

1. [Slack API 페이지](https://api.slack.com/apps) 접속 → **Create New App** → **From scratch**
2. 앱 이름 입력 (예: "Content OS Alert") → 워크스페이스 선택
3. 좌측 메뉴 **Incoming Webhooks** → **Activate Incoming Webhooks** ON
4. **Add New Webhook to Workspace** → 알림 받을 채널 선택 → **Allow**
5. 생성된 Webhook URL 복사 (`https://hooks.slack.com/services/...`)

### 2. GitHub Secret에 URL 저장

1. 리포의 **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. Name: `SLACK_WEBHOOK_URL`
4. Value: 위에서 복사한 URL → **Add secret**

### 3. 동작 확인

- 매일 KST 09:00에 자동 실행 (스케줄은 `.github/workflows/stagnation-alert.yml`에서 변경 가능)
- 수동 실행: 리포의 **Actions** 탭 → **Stagnation Alert** → **Run workflow**

### 알림되는 상황

- 🚨 페이즈 정체 임계 초과 BET이 있는 경우
- ★ 졸업 가능 (L1~L3 모두 완료) BET이 있는 경우
- 📊 매주 월요일은 주간 요약 함께 발송

---

## 📊 스프레드시트 연동 (TBD)

기존에 작성 중인 시트가 있다면 다음 정보를 알려주면 가장 맞는 연동 방식 셋업을 도와드립니다:

1. 시트에 **어떤 데이터**가 들어있는가? (메트릭? 마일스톤? 팀원 부하?)
2. 시트 **공개** 가능한가, 비공개 유지가 중요한가?
3. **읽기만** 필요한가, OS에서 시트로 쓰기도 필요한가?

연동 방식 종류:
- **A) CSV 게시 + fetch** — 가장 간단, 시트 부분 공개됨
- **B) Sheets API + API Key** — 비공개 유지, 인증 약간 복잡
- **C) Apps Script 웹앱 endpoint** — 가장 유연, 양방향 가능
- **D) GitHub Actions 정기 sync** — 시트 → `data.json` 자동 갱신 (권장)

---

## 🧠 운영 OS 컨셉 (참고)

### 위계
- **BET**: 분기 단위 의도. 3개로 압축. 페이즈(L1 Probe → L2 Build → L3 Scale)를 가짐.
- **대 카테고리**: BET 안의 주요 작업 영역. 자체 레벨/마일스톤을 가짐.
- **소 카테고리**: 대 카테고리 안의 구체 프로젝트. 자체 레벨을 가짐.

### 페이즈 (Phase)
- **L1 Probe** — 검증·탐색 (임계 약 45일)
- **L2 Build** — 실행·구축 (임계 약 60-90일)
- **L3 Scale** — 표준화·반복 (임계 약 120일+)

### 정체 트리거
- 페이즈에 임계 초과 머무르면 시각적 경고 + 슬랙 알림
- "왜 안 움직이는가?"를 회의에서 자동 안건화

### 졸업 (Graduation)
- BET의 L1~L3가 모두 완료되면 **졸업 가능** 상태
- 졸업 시 Sustain 트랙으로 이관 + 회고 메모 남길 수 있음
- "베팅 = 1회성 분기 목표"가 아닌 "다단계 자산화 트랙"으로 의미 확장

---

## 🔧 트러블슈팅

| 증상 | 원인 / 해결 |
|---|---|
| 사이트 열어도 빈 화면 | `data.json` 없거나 JSON 문법 오류. 브라우저 개발자 도구 콘솔 확인 |
| 변경사항이 다른 디바이스에서 안 보임 | 클릭 변경은 localStorage라 디바이스 한정. 영구 반영은 `data.json` 직접 수정 |
| 슬랙 알림 안 옴 | Actions 탭에서 실행 로그 확인. Secret 이름이 정확히 `SLACK_WEBHOOK_URL`인지 |
| 페이지가 자동으로 갱신 안 됨 | GitHub Pages는 푸시 후 1-2분 캐시. 브라우저 강력 새로고침 (`Cmd+Shift+R`) |

---

## 📜 라이선스 / 작성

내부 운영용. 외부 공유 시 메트릭은 더미로 교체 권장.
