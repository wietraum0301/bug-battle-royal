# 🪂 BUG ROYALE — GitHub Copilot × G-STAR Developer Night

G-STAR 기간 해운대 **GitHub Copilot 팝업 네트워킹** 방문객 유치를 위한 모바일 웹 게임입니다.
단일 HTML 파일(빌드 불필요)로 GitHub Pages에 바로 배포할 수 있습니다.

| 날짜 | 시간 | 장소 |
|---|---|---|
| 11/19(목) | 17:00–22:00 | 올데이라운지 · 부산 해운대구 센텀2로 25, 106호 |
| 11/20(금) | 17:00–22:00 | 리리코이 · 부산 해운대구 해운대해변로298번길 24, 1층 1-19호 |

🍺 맥주 · 🌮 기본 타코 · 🫓 퀘사디아 무료 — **기업 인증 방문객 only**

## 게임 흐름
1. **정보 입력** — 이름 · 회사 · 직무 · 회사 이메일(네이버/다음/지메일 등 개인 메일 거절) · 휴대폰(선택) · 방문일 · 개인정보 동의
2. **MAIN GAME · PiC MAN (픽셀 아케이드)** — BUG 잡기 GO 버튼 → Copilot 고글을 쓴 PiC MAN이 1.7초 동안 미로를 달려 멈춘 자리의 버그 알림을 냠! → *WINNER WINNER TACO DINNER!*
3. **쿠폰 발급** — 화면 쿠폰 + 회사 이메일로 쿠폰·행사장 주소·시간 발송

| 등급 | 보급품 | 확률 |
|---|---|---|
| P0 Critical | 하몽&멜론 치즈플래터 (33,000원) | 8% |
| P1 Major | 소세지플래터&감자튀김 | 30% |
| P2 Minor | 칵테일 · 하이볼 · 수제맥주 택1 | 62% |

- 쿠폰은 **이메일당 행사일(11/19, 11/20)별 1장**. 재플레이는 연습 경기로 처리되고 **이미 당첨된 쿠폰을 메일로 재발송**합니다.
- 쿠폰 코드 형식: `GSTAR-19-XXXX-XXXX` (날짜 포함)

## 폴더 구성
```
gstar-bug-royale/
├─ index.html                 # 게임 본체 (HTML·CSS·JS 단일 파일, 외부 의존성 없음)
├─ .nojekyll                  # GitHub Pages Jekyll 처리 비활성화
├─ README.md
└─ docs/
   ├─ power-automate-flow.md  # 메일 발송 / 중복 방지 흐름 구성 가이드
   └─ request-schema.json     # 흐름 트리거 요청 본문 JSON 스키마
```

## 설정 변경
`index.html` 안의 `const CONFIG = { ... }` 블록만 수정하면 됩니다.

| 키 | 설명 |
|---|---|
| `SESSIONS` | 행사 날짜 · 장소 · 주소 |
| `MAIL_ENDPOINT` | Power Automate "HTTP 요청을 받은 경우" 트리거 URL (비우면 데모 모드) |
| `PRIZES` | 보급품 · 버그 등급 · 당첨 가중치(`weight`) |
| `BLOCKED_DOMAINS` | 거절할 개인 메일 도메인 |
| `ONE_COUPON_PER_DAY` | 이메일 × 행사일 1장 제한 |

## GitHub Pages 배포
1. GitHub에서 새 저장소 생성 (예: `gstar-bug-royale`, **Public**)
2. 이 폴더의 파일을 저장소 루트에 업로드 (`Add file → Upload files`, `.nojekyll` 포함) 후 Commit
3. `Settings → Pages → Build and deployment` → Source: **Deploy from a branch**, Branch: **main / (root)** → Save
4. 1~2분 후 `https://<계정>.github.io/gstar-bug-royale/` 로 접속 → 이 주소로 QR 코드 제작

CLI:
```bash
cd gstar-bug-royale
git init && git add . && git commit -m "BUG ROYALE for G-STAR Developer Night"
git branch -M main
git remote add origin https://github.com/<계정>/gstar-bug-royale.git
git push -u origin main
```

## ⚠️ 보안 참고
- `MAIL_ENDPOINT` 에는 서명(`sig=`)이 포함되어 있어 **공개 저장소에 올리면 누구나 볼 수 있습니다.** 흐름에서 개인 메일 차단 · 이메일×날짜 중복 검증을 켜 두고, **11/20 행사 종료 후 흐름을 끄거나 URL을 재발급**하세요.
- 중복 발급 방지는 기기(브라우저) 단위 1차 검증 + 흐름(SharePoint 목록) 2차 검증 구조입니다. 2차 검증은 [`docs/power-automate-flow.md`](docs/power-automate-flow.md) 참고.
- 개인정보(이름·회사·이메일·연락처)는 행사 종료 후 1년 이내 파기 안내가 포함되어 있습니다.
