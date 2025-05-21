
# 백엔드 코드 구조

## 데이터베이스 스키마 (db/schema.sql)

```sql
-- games 테이블
CREATE TABLE games (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  meta_score INT,
  user_score FLOAT,
  release_date DATE,
  image_url TEXT
);

-- game_reviews 테이블
CREATE TABLE game_reviews (
  id INT AUTO_INCREMENT PRIMARY KEY,
  game_id INT NOT NULL,
  nickname VARCHAR(100),
  rating FLOAT,
  content TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE
);

-- posts 테이블
CREATE TABLE posts (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  author VARCHAR(100),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- post_comments 테이블
CREATE TABLE post_comments (
  id INT AUTO_INCREMENT PRIMARY KEY,
  post_id INT NOT NULL,
  nickname VARCHAR(100),
  comment TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
);

-- users 테이블 (회원가입용)
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  password VARCHAR(255),
  signup_method VARCHAR(50) NOT NULL DEFAULT 'manual',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

## 설정 파일 (config/db\_connect.php)

```php
<?php
$host = 'localhost';
$db   = 'metacritic';
$user = 'root';
$pass = '';
$charset = 'utf8mb4';

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
     $pdo = new PDO($dsn, $user, $pass, $options);
} catch (\PDOException $e) {
     throw new \PDOException($e->getMessage(), (int)$e->getCode());
}
?>
```

## API 엔드포인트 (api 폴더)

각 API 파일은 `/config/db_connect.php`를 공통으로 사용하며, JSON 형태로 결과를 반환합니다.

* **get\_games.php** – 모든 게임 목록을 반환
* **get\_game.php** – 특정 게임 상세정보 반환
* **add\_review\.php** – 리뷰 추가
* **get\_reviews.php** – 게임별 리뷰 목록
* **signup\_manual.php** – 수동 회원가입
* **signup\_email.php** – 이메일 인증 회원가입
* **update\_games.php** – RAWG API를 통해 게임 데이터 수집 및 업데이트

> 전체 PHP 코드는 [Backend Readme 문서](backend_README) 또는 각각의 파일에서 확인할 수 있습니다.

## 서버 설정 (.htaccess)

```apacheconf
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.html [QSA,L]
```

---

> 💡 PHP 파일별 전체 코드는 실제 백엔드 폴더에서 확인 가능하며, 테스트 시 Postman 또는 브라우저에서 직접 요청할 수 있습니다.
