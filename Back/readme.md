# 백엔드 코드 구조

## 데이터베이스 스키마 (db/schema.sql)

-- games 테이블
```
CREATE TABLE `games` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `description` text,
  `meta_score` int(11) DEFAULT NULL,
  `user_score` float DEFAULT NULL,
  `release_date` date DEFAULT NULL,
  `image_url` text,
  PRIMARY KEY (`id`),
  KEY `title` (`title`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

-- game_reviews 테이블
```
CREATE TABLE `game_reviews` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `game_id` int(11) NOT NULL,
  `nickname` varchar(100) DEFAULT NULL,
  `rating` float DEFAULT NULL,
  `content` text,
  `created_at` datetime DEFAULT current_timestamp(),
  PRIMARY KEY (`id`),
  KEY `game_id` (`game_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

-- users 테이블 
```
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(100) NOT NULL,
  `password` varchar(255) DEFAULT NULL,
  `signup_method` varchar(50) NOT NULL DEFAULT 'manual',
  `created_at` datetime DEFAULT current_timestamp(),
  `role` enum('user','admin') NOT NULL DEFAULT 'user',
  `nickname` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```

## 설정 파일 (config/db_connect.php)

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

### api/get_games.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$sql = "SELECT * FROM games";
$stmt = $pdo->query($sql);
$games = $stmt->fetchAll();

echo json_encode($games);
?>
```

### api/get_game.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$id = isset($_GET['id']) ? intval($_GET['id']) : 0;
if ($id <= 0) {
    http_response_code(400);
    echo json_encode(['error' => '잘못된 요청']);
    exit;
}

$stmt = $pdo->prepare("SELECT * FROM games WHERE id = ?");
$stmt->execute([$id]);
$game = $stmt->fetch();

if ($game) {
    echo json_encode($game);
} else {
    http_response_code(404);
    echo json_encode(['error' => '게임을 찾을 수 없습니다']);
}
?>
```

### api/add_review.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$data = json_decode(file_get_contents("php://input"), true);

$game_id = intval($data['game_id'] ?? 0);
$nickname = trim($data['nickname'] ?? '');
$rating = floatval($data['rating'] ?? 0);
$content = trim($data['content'] ?? '');

if ($game_id <= 0 || $nickname === '' || $content === '') {
    http_response_code(400);
    echo json_encode(['error' => '입력값이 유효하지 않습니다']);
    exit;
}

$stmt = $pdo->prepare("INSERT INTO game_reviews (game_id, nickname, rating, content, created_at) VALUES (?, ?, ?, ?, NOW())");
$success = $stmt->execute([$game_id, $nickname, $rating, $content]);

echo json_encode(['success' => $success]);
?>
```

### api/get_reviews.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$game_id = isset($_GET['game_id']) ? intval($_GET['game_id']) : 0;
if ($game_id <= 0) {
    http_response_code(400);
    echo json_encode(['error' => '잘못된 요청']);
    exit;
}

$stmt = $pdo->prepare("SELECT * FROM game_reviews WHERE game_id = ? ORDER BY created_at DESC");
$stmt->execute([$game_id]);
$reviews = $stmt->fetchAll();

echo json_encode($reviews);
?>
```

### api/signup_manual.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$data = json_decode(file_get_contents("php://input"), true);

$email = trim($data['email'] ?? '');
$password = trim($data['password'] ?? '');

if (!filter_var($email, FILTER_VALIDATE_EMAIL) || strlen($password) < 4) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => '유효한 이메일/비밀번호 필요']);
    exit;
}

$stmt = $pdo->prepare("SELECT id FROM users WHERE email = ?");
$stmt->execute([$email]);
if ($stmt->fetch()) {
    echo json_encode(['success' => false, 'message' => '이미 존재하는 이메일']);
    exit;
}

$hashed = password_hash($password, PASSWORD_DEFAULT);
$stmt = $pdo->prepare("INSERT INTO users (email, password, signup_method) VALUES (?, ?, 'manual')");
$success = $stmt->execute([$email, $hashed]);

echo json_encode(['success' => $success]);
?>
```

### api/signup_email.php
```php
<?php
require_once __DIR__ . '/../config/db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$data = json_decode(file_get_contents("php://input"), true);

$email = trim($data['email'] ?? '');

if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => '이메일 형식이 올바르지 않음']);
    exit;
}

$stmt = $pdo->prepare("SELECT id FROM users WHERE email = ?");
$stmt->execute([$email]);
if ($stmt->fetch()) {
    echo json_encode(['success' => false, 'message' => '이미 가입된 이메일']);
    exit;
}

// 실제 인증 메일 발송 대신, 지금은 바로 저장
$stmt = $pdo->prepare("INSERT INTO users (email, signup_method) VALUES (?, 'email_auth')");
$success = $stmt->execute([$email]);

echo json_encode(['success' => $success]);
?>
```

### api/update_games.php
```php
<?php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

require_once __DIR__ . '/../config/db_connect.php';

$apiKey = '4182ef1bcc1b4031a914a1077401eb6e'; // RAWG API 키
$totalPages = 3; // 최대 120개 게임 가져오기
$pageSize = 40;

// cURL 함수 정의
function curl_get($url) {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    $response = curl_exec($ch);

    if (curl_errno($ch)) {
        echo '❌ cURL 오류: ' . curl_error($ch);
        curl_close($ch);
        exit;
    }

    curl_close($ch);
    return $response;
}

for ($page = 1; $page <= $totalPages; $page++) {
    $apiUrl = "https://api.rawg.io/api/games?key={$apiKey}&page={$page}&page_size={$pageSize}";
    $response = curl_get($apiUrl);
    $data = json_decode($response, true);

    if (!isset($data['results'])) {
        echo "❌ page $page: 응답에 results 없음<br>";
        echo "<pre>"; print_r($data); echo "</pre>";
        continue;
    }

    echo "✅ page $page: " . count($data['results']) . "개 수신<br>";

    foreach ($data['results'] as $game) {
        $title = $game['name'];
        $meta_score = $game['metacritic'] ?? null;
        $user_score = $game['rating'] ?? null;
        $image_url = $game['background_image'] ?? 'https://via.placeholder.com/300x160?text=No+Image';

        // 상세 정보 가져오기 (description)
        $detailUrl = "https://api.rawg.io/api/games/{$game['id']}?key={$apiKey}";
        $detailData = json_decode(curl_get($detailUrl), true);
        $description = $detailData['description_raw'] ?? '';

        // INSERT 또는 UPDATE
        $stmt = $pdo->prepare("INSERT INTO games (title, description, meta_score, user_score, image_url)
                               VALUES (?, ?, ?, ?, ?)
                               ON DUPLICATE KEY UPDATE
                               description = VALUES(description),
                               meta_score = VALUES(meta_score),
                               user_score = VALUES(user_score),
                               image_url = VALUES(image_url)");

        if ($stmt->execute([$title, $description, $meta_score, $user_score, $image_url])) {
            echo "✅ 저장됨: $title<br>";
        } else {
            echo "❌ 저장 실패: $title<br>";
        }
    }

    echo "<hr>";
}

echo "<br>🎉 전체 완료!";
?>
```

## 서버 설정 (.htaccess)

```
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.html [QSA,L]
```

## (tasks.json)
```
{
    "version": "2.0.0",
    "tasks" : [
        {
            "label": "Run PHP",
            "type": "shell",
            "command": "경로",
            "args": [
                "${file}"
            ],
            "group": {
                "kind": "build",
                "inDefault": true
            },
            "problemMatcher": []
        }
    ]
}
```

