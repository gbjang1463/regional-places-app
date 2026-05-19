# 지역별 장소 + 팀에이전트 앱

메모.xlsx 데이터를 기반으로 지역별 장소를 보여주고, 3명의 AI 팀에이전트가 추천을 도와주는 단일 HTML 웹앱입니다.

## ✨ 특징

- **외부 의존성 0** — 단일 HTML 파일, 서버/빌드 도구 불필요
- **지역 필터** — 전체 / 충북 / 경기도
- **3명의 팀에이전트** — 장소 선택 시 역할별 분석 제공
  - 🧭 여행 플래너 — 동선·일정 추천
  - 🏞️ 로컬 가이드 — 지역 특색·맛집
  - 🛡️ 안전 체크 — 날씨·주의사항
- **🔐 구글 로그인** — Google Identity Services 기반 (백엔드 불필요). 설정은 [GOOGLE_LOGIN_SETUP.md](GOOGLE_LOGIN_SETUP.md) 참고
- **반응형 디자인** — 모바일/데스크탑 모두 지원
- **라이트 테마**

## 📍 등록된 장소 (3곳)

| 도 | 지역 | 장소 |
|---|---|---|
| 충북 | 도대리 | 용소폭포 |
| 경기도 | 포천 | 도마치 |
| 경기도 | 남양주 | 청학밸리리조트 |

## 🚀 실행 방법

`index.html` 파일을 더블클릭하거나 브라우저로 열기만 하면 됩니다.

배포된 버전:
- GitHub Pages: <https://gbjang1463.github.io/regional-places-app/>
- Cloud Run: 설정 후 자동 생성 (가이드: [CLOUD_DEPLOY_SETUP.md](CLOUD_DEPLOY_SETUP.md))

## ☁️ Cloud Build / Cloud Run 자동 배포

`main` 브랜치 푸시 시 Cloud Build가 Docker 이미지를 빌드해 Cloud Run에 자동 배포합니다.
초기 설정 가이드는 [CLOUD_DEPLOY_SETUP.md](CLOUD_DEPLOY_SETUP.md)를 참고하세요.

## 🛠️ 기술 스택

- HTML / CSS / Vanilla JavaScript
- 외부 라이브러리·프레임워크 없음

## 📜 라이선스

MIT
