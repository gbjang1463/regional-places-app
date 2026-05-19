# Cloud Build + Cloud Run 배포 설정 가이드

이 저장소를 GitHub → Cloud Build → Cloud Run으로 자동 배포하는 설정 가이드입니다.

## 아키텍처

```
GitHub (main 푸시)
   ↓ 트리거
Cloud Build
   ├─ Docker 이미지 빌드 (nginx:alpine + index.html)
   ├─ Artifact Registry 푸시
   └─ Cloud Run 배포 (asia-northeast3)
                ↓
        https://regional-places-app-xxxx.a.run.app
```

## 사전 준비

- Google Cloud 프로젝트: `regional-places-app` (이미 생성됨)
- GitHub 저장소: `gbjang1463/regional-places-app` (이미 생성됨)
- 결제 계정 연결 필요 (Cloud Run 무료 등급 안에서 동작하지만 활성화는 필수)

## 1. API 활성화

Cloud Console → 검색창에 아래 3개 API를 차례로 검색하여 "사용 설정":

- **Cloud Build API**
- **Cloud Run Admin API**
- **Artifact Registry API**

또는 한 번에:
<https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com,run.googleapis.com,artifactregistry.googleapis.com&project=regional-places-app>

## 2. Artifact Registry 저장소 생성

<https://console.cloud.google.com/artifacts/create-repo?project=regional-places-app>

- **이름**: `regional-places-app`
- **형식**: Docker
- **모드**: 표준
- **위치 유형**: 리전 → **asia-northeast3 (서울)**
- **만들기**

## 3. Cloud Build에 GitHub 연결

<https://console.cloud.google.com/cloud-build/triggers/connect?project=regional-places-app>

1. **소스 → GitHub (Cloud Build GitHub App)** 선택
2. **계속** → GitHub 인증 (`gbjang1463` 계정으로 OAuth 승인)
3. 저장소 선택: `gbjang1463/regional-places-app`
4. **연결**

## 4. Cloud Build 트리거 생성

<https://console.cloud.google.com/cloud-build/triggers/add?project=regional-places-app>

- **이름**: `deploy-on-main`
- **이벤트**: 브랜치로 푸시
- **소스**:
  - 1세대(또는 2세대) → 저장소: 방금 연결한 `gbjang1463/regional-places-app`
  - **브랜치**: `^main$`
- **구성**:
  - **유형**: Cloud Build 구성 파일 (yaml 또는 json)
  - **위치**: 저장소
  - **파일 위치**: `/cloudbuild.yaml` (기본값)
- **서비스 계정**: 기본 Cloud Build 서비스 계정 사용
- **만들기**

## 5. 서비스 계정 권한 부여

Cloud Build 서비스 계정이 Cloud Run에 배포할 수 있도록 권한을 줍니다.

<https://console.cloud.google.com/iam-admin/iam?project=regional-places-app>

기본 Cloud Build 서비스 계정 (`PROJECT_NUMBER@cloudbuild.gserviceaccount.com`)을 찾아 **연필 아이콘 → + 다른 역할 추가**로 아래 역할 부여:

- **Cloud Run 관리자** (`roles/run.admin`)
- **서비스 계정 사용자** (`roles/iam.serviceAccountUser`)
- **아티팩트 레지스트리 작성자** (`roles/artifactregistry.writer`)

## 6. 첫 빌드 실행

방법 A — 자동: main 브랜치에 임의 커밋을 푸시하면 트리거가 동작.

방법 B — 수동:
<https://console.cloud.google.com/cloud-build/triggers?project=regional-places-app>
→ `deploy-on-main` 우측 **실행** 클릭 → 브랜치 `main` 선택 → **트리거 실행**

성공 시:
- Cloud Build 콘솔에서 빌드 로그 확인 가능
- Cloud Run 콘솔에 `regional-places-app` 서비스가 생성됨
- 우측 상단에 서비스 URL 표시 (예: `https://regional-places-app-xxxx-an.a.run.app`)

## 7. Google 로그인 OAuth 출처 추가

Cloud Run URL을 OAuth 클라이언트의 승인된 JavaScript 출처에 추가해야 구글 로그인이 동작합니다.

<https://console.cloud.google.com/auth/clients?project=regional-places-app>

→ `web-prod` 클릭 → **승인된 JavaScript 출처**에 Cloud Run URL 추가:
```
https://regional-places-app-xxxx-an.a.run.app
```
→ 저장

## 비용 안내

- **Cloud Run**: 월 200만 요청까지 무료 (개인 사이드 프로젝트 수준에선 사실상 무료)
- **Cloud Build**: 일 120 빌드분 무료 (이 앱은 빌드 1~2분)
- **Artifact Registry**: 월 0.5GB 무료 (이 이미지는 ~20MB)

## 문제 해결

| 증상 | 원인 / 해결 |
|---|---|
| 트리거가 실행 안 됨 | GitHub App이 저장소에 액세스 권한 있는지 확인 (3단계) |
| `permission denied` on Cloud Run | 5단계 권한 누락 |
| `manifest unknown` | Artifact Registry 저장소 미생성 (2단계) |
| 빌드는 성공했는데 사이트 접속 안 됨 | Cloud Run 콘솔에서 "할당량 없음"·"리전 다름" 확인 |
| Google 로그인 실패 (origin not allowed) | 7단계 — Cloud Run URL 출처 추가 |

## 참고

- `cloudbuild.yaml`: 빌드/푸시/배포 3단계 정의
- `Dockerfile`: nginx:alpine 기반 정적 호스팅
- `nginx.conf`: 포트 8080 (Cloud Run 표준), gzip 활성화, SPA fallback
- `.gcloudignore` / `.dockerignore`: 빌드 컨텍스트 최소화
