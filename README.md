# Skala Stock Vue

주식 거래 시뮬레이션 웹 애플리케이션의 프론트엔드 부분입니다. [skala-stock-server](https://github.com/zinsile/skala-stock-server) 프로젝트와 함께 사용되도록 설계되었습니다.

## 주요 기능

### 플레이어 관리
- 플레이어 생성 및 로그인
- 플레이어 자금 확인 및 추가
- 보유 주식 확인
- 플레이어 탈퇴

### 주식 거래
- 주식 목록 조회
- 새로운 주식 종목 추가
- 주식 매수 및 매도

## 기술 스택

- Vue.js 3 (Composition API)
- Axios (HTTP 클라이언트)
- CSS (컴포넌트별 Scoped 스타일링)

## 프로젝트 구조

```
skala-stock-vue/
├── public/
├── src/
│   ├── assets/         # 이미지, 폰트 등 정적 자원
│   ├── components/     # Vue 컴포넌트
│   │   ├── PlayerList.vue  # 플레이어 목록 및 관리
│   │   └── StockList.vue   # 주식 목록 및 거래
│   ├── App.vue         # 메인 애플리케이션 컴포넌트
│   ├── main.js         # 애플리케이션 진입점
│   └── style.css       # 전역 스타일
├── .gitignore
├── index.html          # 메인 HTML 페이지
├── package.json        # 프로젝트 의존성 및 스크립트
├── package-lock.json
├── README.md
└── vite.config.js      # Vite 설정
```

## 설치 및 실행

### 요구사항
- Node.js 16 이상
- npm 또는 yarn
- [skala-stock-server](https://github.com/zinsile/skala-stock-server) 백엔드 서버 (v3 브랜치)

### 설치 방법

```bash
# 프로젝트 클론
git clone https://github.com/zinsile/skala-stock-vue.git
cd skala-stock-vue

# 의존성 설치
npm install
# 또는
yarn install
```

### 개발 서버 실행

```bash
npm run dev
# 또는
yarn dev
```

기본적으로 `http://localhost:5173`에서 애플리케이션을 확인할 수 있습니다.

### 프로덕션 빌드

```bash
npm run build
# 또는
yarn build
```

빌드된 파일은 `dist` 디렉토리에 생성됩니다.

## 백엔드 연결

이 프론트엔드는 기본적으로 `http://localhost:8080`에서 실행 중인 백엔드 서버와 통신하도록 설정되어 있습니다. 백엔드 URL을 변경하려면 각 컴포넌트의 axios 요청 URL을 수정하세요.

## 사용 방법

1. 백엔드 서버(skala-stock-server)를 먼저 실행합니다.
2. 프론트엔드 개발 서버를 실행합니다.
3. 브라우저에서 `http://localhost:5173`에 접속합니다.
4. 플레이어 이름을 입력하여 로그인하거나 새 플레이어를 생성합니다.
5. 주식을 매수/매도하거나 새로운 주식을 추가할 수 있습니다.