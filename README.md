# Final PY API

트렌드 기반 AI 콘텐츠 생성 및 발행 자동화 백엔드

<br>

## 프로젝트 소개

Final PY API는 실시간 트렌드 키워드를 수집하고, 이커머스 상품 데이터를 매칭한 뒤, LLM을 활용해 홍보 콘텐츠를 생성하고 발행하는 Python 기반 AI 백엔드 서비스입니다.

Google Trends, 쿠팡, 네이버 쇼핑, 싸다구(Ssadagu.kr) 등 다양한 채널에서 트렌드와 상품 데이터를 수집하고, 수집된 데이터를 LLM 기반 키워드 정제 및 상품 연관도 평가에 활용합니다. 이후 네이버 블로그에 적합한 콘텐츠를 생성하고 업로드/발행하는 자동화 파이프라인을 제공합니다.

저는 Playwright 기반 크롤링 엔진, 비동기 FastAPI 구조 전환, LLM 기반 키워드 정제 및 연관도 평가, LLM 설정 관리 API, Docker/AWS 배포 및 CI/CD 자동화 등 핵심 기능과 인프라 전반을 주도적으로 구현했습니다.

<br>

## 핵심 기여 요약

- Playwright 기반 Google Trends 실시간 키워드 수집 크롤러 구축
- 쿠팡, 네이버 쇼핑, 싸다구 상품 리스트 및 상세 정보 스크래핑 구현
- Jupyter Notebook 기반 동기식 레거시 코드를 FastAPI 비동기 서비스 구조로 전환
- LLM 기반 트렌드 키워드 정제 및 쇼핑몰 검색어 변환 파이프라인 구축
- OCR 및 Vision API 기반 상품 유사도 분석 설계
- `RelevanceService` 기반 키워드-상품 연관도 평가 시스템 구현
- LLM 설정 조회/수정/등록 API 및 설정 도메인 구조 구현
- Playwright/Chromium 실행을 위한 Docker 기반 컨테이너 환경 구성
- AWS ECR 빌드 스크립트 및 GitHub Actions 기반 CI/CD 배포 자동화
- 배포 환경변수 유실 문제 해결 및 안정적인 배포 파이프라인 개선

<br>

## 개인 담당 및 기여 내용

### 1. 다중 채널 트렌드 및 데이터 수집 엔진 구축

실시간 시장 트렌드를 파악하고 홍보 타겟 상품을 매칭하기 위해 Playwright 기반 크롤링 엔진을 설계하고 지속적으로 고도화했습니다.

- Google Trends 페이지에서 실시간 인기 검색어 약 80개를 안정적으로 추출하는 크롤러를 구현했습니다.
- UI 문구, 중복 텍스트, 불필요한 메타데이터가 키워드에 포함되지 않도록 필터링 로직을 추가했습니다.
- 정제된 트렌드 키워드와 연관된 상품 정보를 확보하기 위해 쿠팡, 네이버 쇼핑, 싸다구 사이트 대상 크롤러를 개발했습니다.
- 검색어별 상위 상품 리스트의 상품명, 가격, 썸네일 등을 추출하고 표준 JSON 형태로 반환하는 구조를 설계했습니다.
- 단순 리스트 조회를 넘어 상품 상세 페이지의 상세 스펙과 가격 정보를 가져오는 심층 크롤링을 구현했습니다.

### 2. 동기식 레거시 스크립트 리팩토링 및 아키텍처 고도화

실험 단계의 Jupyter Notebook 기반 크롤링 코드를 상용 서비스에 가까운 비동기 웹 애플리케이션 구조로 전면 개편했습니다.

- 기존 `.ipynb` 형태의 동기식 단일 셀 크롤링 스크립트를 기능별로 분리했습니다.
- 수집, 가공, 반환 책임을 구분하고 FastAPI의 `app/` 폴더 하위 표준 Python 모듈로 재구성했습니다.
- API 엔드포인트는 얇게 유지하고, 외부 호출과 비즈니스 로직은 `app/services` 계층으로 이동했습니다.
- 비동기 처리 구조를 적용해 대규모 배치 작업과 API 요청 처리 효율성을 높였습니다.

### 3. LLM 기반 지능형 데이터 정제 및 연관도 평가 엔진 개발

수집된 트렌드 키워드가 실제 커머스 환경과 쇼핑몰 성격에 맞게 변환되고, 적합한 상품과 매칭되도록 LLM 기반 정제 및 평가 로직을 통합했습니다.

- 원시 트렌드 키워드를 중국 생산품 직구 쇼핑몰에 적합한 실제 이커머스 검색어(`real_keyword`)로 변환하는 프롬프트 파이프라인을 구축했습니다.
- LLM이 검색어 변환 결과와 함께 선택 사유(`reason`)를 제안하도록 설계했습니다.
- 상품명, 가격, 썸네일 이미지, 상품 상세정보의 OCR 텍스트를 활용해 상품 간 유사도를 분석하는 기능을 설계했습니다.
- 이미지 분석에는 Vision API를 도입해 텍스트 기반 판단의 한계를 보완했습니다.
- 선택된 키워드와 상품 상세 정보가 마케팅에 적합한지 LLM이 0.0~1.0 사이 점수로 평가하는 `RelevanceService`를 구축했습니다.
- `WriteService` 오케스트레이션에서 연관도 임계치에 미달하는 상품은 최대 5회까지 재탐색하도록 구성해 광고 글 품질을 개선했습니다.

### 4. LLM 설정 관리 기능 및 백엔드 연동 API 구현

시스템 내부의 LLM 작동 방식과 채널 속성을 동적으로 제어할 수 있도록 설정 도메인을 설계하고 API를 개발했습니다.

- LLM 설정의 모델명, temperature, 시스템 프롬프트 등을 조회하고 수정할 수 있는 API 레이어를 구현했습니다.
- LLM 설정 관련 Domain, DTO, Controller, Service, Mapper 계층을 구성했습니다.
- 설정 조회/수정 흐름에 대한 테스트 코드를 작성했습니다.
- 설정 등록 기능을 추가하고, 수정 API를 RESTful 규격에 맞게 `PUT` 메서드로 전환했습니다.

### 5. Docker/AWS 인프라 및 CI/CD 자동화 구축

Python 백엔드와 AI 서비스 환경을 컨테이너화하고, 클라우드 환경에 안정적으로 배포할 수 있도록 인프라와 자동화 파이프라인을 구축했습니다.

- Playwright, Chromium 등 브라우저 자동화 의존성이 컨테이너 내부에서 동작하도록 `Dockerfile`을 작성했습니다.
- 컨테이너 시작 시 가상 디스플레이 환경이 안정적으로 구동되도록 `docker-entrypoint.sh`를 구성했습니다.
- AWS 배포를 위한 컨테이너 기반 실행 구조와 빌드 설정을 정리했습니다.
- 운영 환경에서 프론트엔드가 백엔드 API를 정상적으로 호출하도록 엔드포인트 연결과 도메인 불일치 문제를 해결했습니다.
- 트렌드 화면 등에서 API가 무한 호출되던 프론트-백 연동 병목 버그를 수정했습니다.
- AWS ECR 빌드 스크립트와 인프라 설정을 추가했습니다.
- 메인 브랜치 PR 머지 후 백엔드 및 Python AI 서비스가 자동 빌드/배포되도록 CI/CD 파이프라인을 자동화했습니다.
- 자동 배포 시 AWS 콘솔에 수동 입력한 환경변수가 유실되던 문제를 발견하고, 배포 정의서 및 파이프라인 내부에 명시적으로 주입되도록 개선했습니다.

<br>

## 기술적 문제 해결 및 최적화

### 1. 크롤링 데이터 품질 개선

실시간 트렌드와 쇼핑몰 페이지는 UI 구조와 노출 문구가 자주 변경되기 때문에, 단순 텍스트 추출만으로는 안정적인 데이터 확보가 어려웠습니다.

- Google Trends 수집 결과에 UI 문구나 불필요한 메타데이터가 섞이는 문제를 필터링 로직으로 개선했습니다.
- 쇼핑몰별 HTML 구조 차이를 고려해 상품명, 가격, 썸네일 등 공통 필드를 표준 JSON으로 정규화했습니다.
- 크롤링 결과가 LLM 정제와 연관도 평가 단계에서 바로 활용될 수 있도록 데이터 반환 구조를 통일했습니다.

### 2. LLM 기반 상품 매칭 품질 개선

트렌드 키워드가 실제 쇼핑몰 검색어로 바로 사용되기 어려운 문제를 해결하기 위해 LLM 기반 정제 단계를 도입했습니다.

- 원시 키워드를 쇼핑몰 상품 검색에 적합한 검색어로 변환했습니다.
- 연관도 점수가 낮은 상품을 반복적으로 재탐색하는 흐름을 적용해 최종 콘텐츠 품질을 높였습니다.

### 3. 컨테이너 크롤링 환경 안정화

Playwright와 Chromium은 로컬 환경과 서버 환경의 의존성 차이에 민감하므로, Docker 내부에서 동일하게 실행되는 환경을 구성했습니다.

- Chromium 및 시스템 의존성을 Docker 이미지에 포함했습니다.
- Docker Compose 환경에서 API 서버와 테스트 컨테이너를 동일한 구조로 실행할 수 있게 구성했습니다.

### 4. 배포 안정성 개선

자동 배포 과정에서 수동으로 입력한 환경변수가 유실되는 문제를 발견하고, 배포 정의와 CI/CD 파이프라인을 개선했습니다.

- 배포에 필요한 환경변수와 시크릿 주입 방식을 명시적으로 정리했습니다.
- AWS ECR 빌드 및 배포 스크립트를 구성했습니다.
- PR 머지 이후 자동으로 빌드와 배포가 이어지는 흐름을 구축했습니다.

<br>

## 프로젝트 주요 기능

### 트렌드 수집

- Google Trends 실시간 키워드 수집
- 인기 검색어 필터링 및 정제
- 트렌드 키워드 API 제공

### 상품 데이터 수집

- 쿠팡 상품 리스트 스크래핑
- 네이버 쇼핑 상품 리스트 및 상세 정보 스크래핑
- 싸다구 상품 리스트 스크래핑
- 상품명, 가격, 썸네일, 상세 스펙 수집
- 표준 JSON 응답 구조 제공

### 키워드 정제

- LLM 기반 원시 트렌드 키워드 변환
- 쇼핑몰 검색어 생성

### 상품 연관도 평가

- 키워드와 상품 상세 정보 간 연관도 점수화
- LLM 기반 0.0~1.0 정밀 평가
- OCR 텍스트 및 이미지 정보 활용
- 기준 점수 미달 시 상품 재탐색

### 콘텐츠 생성

- 홍보 문구 생성
- 네이버 블로그용 콘텐츠 생성
- LangGraph 기반 글쓰기 오케스트레이션

### 업로드 및 발행

- 콘텐츠 업로드 API
- 네이버 블로그 발행 연동
- 채널별 콘텐츠 포맷 처리

### 설정 관리

- LLM 설정 조회
- LLM 설정 수정
- 모델명, temperature, 시스템 프롬프트 관리
- 채널별 생성 타입 관리

<br>

## 기술 스택

### Backend

- Python 3.11
- FastAPI
- Pydantic
- LangGraph
- Pytest

### Crawling

- Playwright
- Chromium

### AI

- OpenAI API
- Vision API
- LLM Prompt Pipeline
- OCR 기반 텍스트 활용

### DevOps

- Docker
- Docker Compose
- AWS ECR
- GitHub Actions

### External Services

- Google Trends
- Coupang
- Naver Shopping
- Ssadagu.kr
- Naver Blog

<br>

## 프로젝트 구조

```text
Final-PY-Fork/
├── app/
│   ├── main.py                  # FastAPI 애플리케이션 진입점
│   ├── config.py                # 환경변수 및 설정 관리
│   ├── api/
│   │   ├── routes.py            # 공통 API 라우트
│   │   └── v1/
│   │       ├── router.py        # v1 API 라우터
│   │       └── endpoints/       # 기능별 엔드포인트
│   ├── services/                # 크롤링, LLM, 발행 등 비즈니스 로직
│   ├── schemas/                 # Pydantic 요청/응답 모델
│   ├── prompts/                 # LLM 프롬프트 템플릿
│   └── flows/
│       └── write_graph.py       # LangGraph 기반 글쓰기 오케스트레이션
├── tests/                       # Pytest 테스트 코드
├── .github/workflows/           # CI/CD 워크플로우
├── Dockerfile                   # Playwright/Chromium 실행 이미지
├── docker-compose.yml           # 로컬 API 및 테스트 컨테이너 구성
├── docker-entrypoint.sh         # Xvfb 기반 실행 엔트리포인트
├── requirements.txt             # Python 의존성
└── README.md
```

<br>

## 아키텍처

프로젝트는 FastAPI 기반 레이어드 아키텍처를 따릅니다.

- API Layer: HTTP 요청/응답 처리 및 라우팅
- Schema Layer: Pydantic 기반 요청/응답 데이터 검증
- Service Layer: 크롤링, LLM 호출, 연관도 평가, 업로드/발행 등 비즈니스 로직 처리
- Prompt Layer: LLM 프롬프트 템플릿 관리
- Flow Layer: LangGraph 기반 콘텐츠 생성 오케스트레이션
- Config Layer: 환경변수 및 외부 서비스 설정 관리

<br>

## 실행 방법

Docker로 실행하는 것을 권장합니다.

1. `.env.example`을 참고해 `.env` 파일을 작성합니다.
2. 아래 명령으로 컨테이너를 빌드/실행합니다.

```bash
docker compose up --build
```

실행 후 기본 헬스 체크는 `GET /health`로 확인할 수 있습니다.

<br>

## 로컬 개발

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Windows PowerShell에서는 가상환경 활성화 명령을 다음과 같이 사용할 수 있습니다.

```powershell
.\.venv\Scripts\Activate.ps1
```

<br>

## 테스트

```bash
python -m pytest
```

Docker 환경에서는 다음 명령으로 테스트를 실행할 수 있습니다.

```bash
docker compose run --rm test
```

<br>

## 환경 변수

필요한 환경 변수는 `.env.example`을 기준으로 설정합니다. API 키, 배포 시크릿, 채널 토큰 등 민감 정보는 `.env` 또는 배포 파이프라인의 시크릿 관리 기능을 통해 주입하며, 저장소에는 커밋하지 않습니다.
