# 🎮 게임 리뷰 웹사이트 프로젝트 - reactproject

게임 정보를 확인하고 사용자들이 리뷰를 작성할 수 있는 웹 애플리케이션입니다.  
프론트엔드는 **React**, 백엔드는 **PHP + MySQL**로 구성되어 있으며, 간단한 로그인 기능과 리뷰 CRUD 기능을 제공합니다.

---

## 📁 프로젝트 구조

```
reactproject/
├── frontend/       # React 기반 프론트엔드
│   ├── src/
│   │   ├── components/   # 재사용 가능한 UI 컴포넌트
│   │   ├── pages/        # 페이지별 라우팅 구성
│   │   └── App.jsx       # 메인 컴포넌트
│   ├── public/           # 정적 리소스
│   └── build/            # 빌드 결과물
├── backend/        # PHP 기반 백엔드
│   ├── api/        # API 엔드포인트 (게임/유저/리뷰 처리)
│   ├── db_connect.php    # DB 연결
│   └── static/     # 정적 CSS 리소스
```

---

## ✨ 주요 기능

- 🔍 게임 목록 조회
- 📋 게임 상세 정보 보기
- ✍ 리뷰 작성 및 조회
- 👤 사용자 회원가입 및 로그인
- 🔐 관리자 기능 (권한에 따른 리뷰 삭제 및 유저 관리)

---

## 🛠️ 기술 스택

### ✅ 프론트엔드
- React (Hooks 기반)
- React Router DOM
- Fetch API
- 기본 CSS 스타일링

### ✅ 백엔드
- PHP (절차지향 스타일)
- MySQL (InnoDB)
- RESTful API 설계

---

## ⚙️ 설치 방법

1. 📥 리포지토리 클론
```bash
git clone https://github.com/couque0319/reactproject.git
cd reactproject
```

2. 🛠️ 프론트엔드 설정
```bash
cd frontend
npm install
npm start
```

3. 🗄️ 백엔드 설정
- `backend/db_connect.php` 파일을 열어 본인의 DB 설정값을 입력하세요.
- MySQL에 [backend/README.md](./backend/README.md)에 포함된 테이블 스키마를 반영하세요.

4. 🔄 라우팅 설정 (.htaccess)
- Apache 사용 시 SPA 라우팅 문제 방지를 위한 `.htaccess` 설정이 포함되어 있습니다.

---

## 📡 주요 API 엔드포인트 (백엔드)

| 메소드 | 경로                         | 설명             |
|--------|------------------------------|------------------|
| GET    | `/api/get_games.php`         | 전체 게임 목록 조회 |
| GET    | `/api/get_game.php?id=1`     | 특정 게임 정보 조회 |
| POST   | `/api/add_review.php`        | 리뷰 작성        |
| GET    | `/api/get_reviews.php?game_id=1` | 게임별 리뷰 조회 |
| POST   | `/api/register.php`          | 회원가입         |
| POST   | `/api/login.php`             | 로그인           |
| GET    | `/api/session_status.php`    | 로그인 세션 확인  |

---

## 📄 라이선스

본 프로젝트는 학습 및 포트폴리오 용도로 자유롭게 활용 가능합니다.
