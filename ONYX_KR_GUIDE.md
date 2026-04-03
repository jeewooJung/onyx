# Onyx - 로컬 시작 가이드

## 📌 Onyx란?

**Onyx**는 LLM(대규모 언어 모델)을 위한 오픈소스 AI 플랫폼입니다. 다음 기능을 제공합니다:

- **RAG (검색 증강 생성)**: 문서 검색 + AI 응답
- **웹 검색**: 최신 정보 수집
- **AI 에이전트**: 맞춤형 지능형 봇 구축
- **코드 실행**: 샌드박스 환경에서 코드 실행
- **음성 모드**: 음성으로 대화
- **아티팩트**: 문서/그래픽 생성

---

## 🛠️ 요구사항

### 필수
- Docker & Docker Compose (또는 개별 설치된 PostgreSQL, Redis)
- 최소 4GB RAM

### 선택사항
- AI API Key (OpenAI, Anthropic 등) - 없어도 로컬 LLM으로 대체 가능

---

## 🚀 빠른 시작 (Docker Compose 사용)

### 1단계: 설정 파일 준비

```bash
cd deployment/docker_compose
cp env.template .env
```

**파일 위치:**
```
onyx/
├── deployment/
│   └── docker_compose/          ← 이 디렉토리에서 작업
│       ├── .env                  ← 생성/수정할 파일
│       ├── env.template
│       ├── docker-compose.yml
│       └── ...
```

### 2단계: 서비스 실행

```bash
docker compose up -d
```

### 3단계: 접속

- **URL**: `http://localhost:3000`
- **기본 로그인**: 
  - 이메일: `admin@example.com`
  - 비밀번호: 첫 접속 시 설정

---

## 💾 기존 PostgreSQL/Redis 사용

기존 데이터베이스와 캐시 서버가 있는 경우:

### 1단계: .env 파일 준비

```bash
cd deployment/docker_compose
cp .env.external-db.example .env
```

**파일 위치:**
```
onyx/
├── deployment/
│   └── docker_compose/              ← 이 디렉토리에서 작업
│       ├── .env.external-db.example ← 참고 파일
│       ├── .env                      ← 생성/수정할 파일 (이곳)
│       ├── env.template
│       ├── docker-compose.yml
│       └── ...
```

**⚠️ 중요:** `.env` 파일은 **반드시** `deployment/docker_compose/` 디렉토리에 있어야 합니다!
- Python은 이 파일을 자동으로 읽지 않음
- Docker Compose가 읽어서 환경 변수로 변환
- 다른 위치에 있으면 설정이 적용되지 않음

### 2단계: 환경 변수 설정

`.env` 파일을 열어서 다음을 수정하세요:

```env
# PostgreSQL 설정
POSTGRES_HOST=your_postgres_host          # 예: localhost, 192.168.1.100, db.example.com
POSTGRES_PORT=5432                         # PostgreSQL 포트 (기본값: 5432)
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=onyx_db                       # 새로 생성될 DB 이름

# Redis 설정
REDIS_HOST=your_redis_host                # 예: localhost, 192.168.1.100, redis.example.com
REDIS_PORT=6379                            # Redis 포트 (기본값: 6379)

# 파일 저장소 설정
FILE_STORE_BACKEND=postgres               # PostgreSQL을 파일 저장소로 사용 (권장)
```

### 주의: Docker 호스트명 설정

**Docker for Desktop (Mac/Windows):** `localhost` 또는 `host.docker.internal` 사용
```env
POSTGRES_HOST=host.docker.internal
REDIS_HOST=host.docker.internal
```

**Docker on Linux:** 호스트의 실제 IP 주소 사용
```bash
# 호스트 IP 확인
hostname -I

# .env 파일에 입력
POSTGRES_HOST=192.168.1.100
REDIS_HOST=192.168.1.100
```

### 3단계: PostgreSQL/Redis 연결 확인

```bash
# PostgreSQL 확인
psql -h POSTGRES_HOST -U POSTGRES_USER -d POSTGRES_DB

# Redis 확인
redis-cli -h REDIS_HOST ping
# 응답: PONG
```

### 4단계: 서비스 실행

```bash
docker compose up -d
```

**Docker가 자동으로 relational_db와 cache 서비스를 건너뜁니다.**

### 5단계: 로그 확인

```bash
# 연결 에러 확인
docker compose logs -f api_server

# 정상 시작 메시지
# "Verifying default connector/credential exist."
```

---

## 📂 .env 파일 위치 정리

### 기본 설정 (env.template 사용)
```bash
# 디렉토리 구조
deployment/docker_compose/
├── env.template          ← 원본 (읽기 전용)
├── .env                  ← 수정할 파일 (이곳에 생성)
└── docker-compose.yml
```

### 기존 DB 사용 (외부 DB용)
```bash
# 디렉토리 구조
deployment/docker_compose/
├── .env.external-db.example    ← 참고용
├── .env                        ← 수정할 파일 (이곳에 생성)
└── docker-compose.yml
```

### ✅ 정확한 경로 확인

```bash
# 프로젝트 루트에서
cd deployment/docker_compose
ls -la .env          # .env 파일 확인
pwd                  # 현재 경로 확인
```

**출력 예시:**
```
$ pwd
/Users/yourname/onyx/deployment/docker_compose
```

---

## ⚙️ 주요 설정

### AI 모델 설정 (선택사항)

**OpenAI 사용:**
```env
GEN_AI_API_KEY=sk-your-openai-key
GEN_AI_MODEL_VERSION=gpt-4o-mini
```

**로컬 LLM 사용 (API key 없음):**
```env
# Ollama 사용
GEN_AI_MODEL_VERSION=ollama
# Ollama 서비스가 실행 중이어야 함
```

### 웹 검색 설정 (선택사항)

```env
# 예시: Serper API 사용
# SERPER_API_KEY=your-serper-key
```

### 인증 설정

```env
# 기본 인증 (username/password)
AUTH_TYPE=basic
USER_AUTH_SECRET=generated_secret_key

# 보안을 위해 다음으로 생성:
# openssl rand -hex 32
```

---

## 📊 시스템 구조

```
Onyx (웹 UI)
    ↓
API Server (백엔드)
    ├── PostgreSQL (데이터 저장소)
    ├── Redis (캐시)
    ├── OpenSearch (문서 검색 인덱스)
    ├── Model Server (Embedding 모델)
    └── MinIO (파일 저장소)
```

**각 컴포넌트 역할:**
- **API Server**: 메인 백엔드 로직
- **PostgreSQL**: 사용자, 채팅, 문서 메타데이터
- **Redis**: 세션, 캐시
- **OpenSearch**: 문서 전체 텍스트 검색 (RAG용)
- **Model Server**: Embedding 생성 (검색 모델)
- **MinIO**: 업로드된 파일 저장

---

## 🔍 RAG 작동 원리

### API Key 없이 작동하는 부분
✅ **검색 (Retrieval)**
- Embedding 모델로 문서 벡터화
- OpenSearch에서 유사한 문서 찾기
- **로컬 실행 - API key 불필요**

### API Key 필요한 부분
❌ **응답 생성 (Generation)**
- LLM으로 최종 답변 생성
- **외부 API 또는 로컬 LLM 필요**

### 해결책
**로컬 LLM 사용으로 API key 제거:**
```bash
# Ollama 설치 및 실행
curl https://ollama.ai/install.sh | sh
ollama run mistral  # 또는 다른 모델
```

---

## 🐛 문제 해결

### PostgreSQL 연결 실패
```bash
# PostgreSQL 서버 확인
psql -h your_postgres_host -U your_db_user -d onyx_db

# Docker 내부에서 테스트
docker exec -it api_server ping your_postgres_host
```

### Redis 연결 실패
```bash
# Redis 접속 확인
redis-cli -h your_redis_host ping
```

### 포트 이미 사용 중
```bash
# 현재 실행 중인 컨테이너 확인
docker ps

# 기존 Onyx 중지
docker compose down
```

### 파일 업로드 실패
- MinIO 설정 확인: `FILE_STORE_BACKEND=postgres`로 변경
- 또는 MinIO 서비스 상태 확인: `docker logs minio`

### .env 파일이 적용되지 않음
```bash
# 1) 파일 위치 확인
pwd  # deployment/docker_compose 에 있어야 함
ls -la .env

# 2) 파일 내용 확인
cat .env | grep POSTGRES_HOST

# 3) Docker 환경변수 확인
docker exec api_server env | grep POSTGRES_HOST
```

✅ **올바른 위치:**
```
/path/to/onyx/deployment/docker_compose/.env
```

❌ **잘못된 위치:**
```
/path/to/onyx/.env              (X)
/path/to/onyx/backend/.env      (X)
/path/to/onyx/deployment/.env   (X)
```

---

## 📝 유용한 명령어

```bash
# 로그 확인
docker compose logs -f api_server

# 서비스 상태 확인
docker compose ps

# 서비스 중지
docker compose down

# 데이터 초기화 (주의!)
docker compose down -v

# 특정 서비스만 재시작
docker compose restart api_server
```

---

## 🔗 유용한 링크

- **공식 문서**: https://docs.onyx.app
- **GitHub 저장소**: https://github.com/onyx-dot-app/onyx
- **커뮤니티**: https://discord.gg/TDJ59cGV2X

---

## 📌 체크리스트

### Docker Compose 사용 시
- [ ] Docker & Docker Compose 설치
- [ ] `cd deployment/docker_compose` 로 이동
- [ ] `env.template` → `.env` 복사
- [ ] `.env` 파일이 `deployment/docker_compose/` 에 있는지 확인
- [ ] `docker compose up -d` 실행
- [ ] `http://localhost:3000` 접속 확인
- [ ] (선택) OpenAI API Key 설정

### 기존 DB 사용 시
- [ ] `cd deployment/docker_compose` 로 이동
- [ ] `.env.external-db.example` → `.env` 복사
- [ ] `.env` 파일이 `deployment/docker_compose/` 에 있는지 확인
- [ ] PostgreSQL 실행 중 확인
- [ ] Redis 실행 중 확인
- [ ] `.env`에서 호스트/포트/비밀번호 수정
- [ ] `docker compose up -d` 실행
- [ ] DB 연결 확인: `docker logs api_server`

---

**Happy Onyx! 🚀**
