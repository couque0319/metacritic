# 게임 리뷰 웹사이트 프로젝트

게임 정보를 표시하고 사용자가 리뷰를 작성할 수 있는 웹 애플리케이션입니다.

## 프로젝트 구조

```
project-root/
├── frontend/       # React 프론트엔드 코드
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── ...
│   └── public/
└── backend/        # PHP 백엔드 코드
    ├── api/        # API 엔드포인트
    ├── config/     # 설정 파일
    └── db/         # 데이터베이스 관련
```

## 기능

- 게임 목록 표시
- 게임 상세 정보 보기
- 리뷰 작성 및 조회
- 사용자 회원가입 및 로그인

## 기술 스택

### 프론트엔드
- React
- React Router
- CSS

### 백엔드
- PHP
- MySQL

## 설치 방법

1. 리포지토리 클론
```bash
git clone https://github.com/yourusername/game-review-website.git
cd game-review-website
```

2. 백엔드 설정
- MySQL 데이터베이스 생성
- `backend/db/schema.sql` 파일의 SQL 명령어로 테이블 생성
- `backend/config/db_connect.php`에서 데이터베이스 연결 정보 설정

3. 프론트엔드 설정
```bash
cd frontend
npm install
npm start
```

4. `.htaccess` 파일로 라우팅 설정

## API 엔드포인트

- `GET /api/get_games.php` - 모든 게임 목록 조회
- `GET /api/get_game.php?id=<id>` - 특정 게임 정보 조회
- `POST /api/add_review.php` - 리뷰 추가
- `GET /api/get_reviews.php?game_id=<id>` - 특정 게임의 리뷰 조회
- `POST /api/signup_manual.php` - 일반 회원가입
- `POST /api/signup_email.php` - 이메일 인증 회원가입
