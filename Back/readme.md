# 🔧 Backend - reactproject

React 기반 게임 리뷰 프로젝트의 **백엔드(PHP API)**입니다.

## 📁 디렉토리 구조

```
backend/
├── .htaccess
├── db_connect.php
├── index.html
├── api/
│   ├── add_review.php
│   ├── ask_ai.php
│   ├── check_username.php
│   ├── db.php
│   ├── delete_game.php
│   ├── delete_review.php
│   ├── get_game.php
│   ├── get_games.php
│   ├── get_reviews.php
│   ├── get_reviews_by_user.php
│   ├── get_users.php
│   ├── login.php
│   ├── logout.php
│   ├── register.php
│   ├── session_status.php
│   ├── signup_manual.php
│   ├── update_games.php
│   ├── update_user_role.php
├── static/
│   └── css/
```

## 📌 주요 설명

- **`db_connect.php`** : DB 연결 설정
- **`api/`** : 게임, 유저, 리뷰 관련 API 핸들러
- **`static/css/`** : 정적 CSS 리소스

## 🧩 PHP API 코드

### `api/add_review.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');
ini_set('display_errors', 1);
error_reporting(E_ALL);

$input = json_decode(file_get_contents("php://input"), true);

$game_id = $input['game_id'] ?? null;
$nickname = $input['nickname'] ?? null;
$rating = $input['rating'] ?? null;
$content = $input['content'] ?? null;

if (!$game_id || !$nickname || !$rating || !$content) {
  echo json_encode(['success' => false, 'message' => '모든 필드를 입력해주세요.']);
  exit;
}

$stmt = $pdo->prepare("INSERT INTO game_reviews (game_id, nickname, rating, content) VALUES (?, ?, ?, ?)");
$success = $stmt->execute([$game_id, $nickname, $rating, $content]);

echo json_encode(['success' => $success]);

```

### `api/ask_ai.php`
```php
<?php
// 에러 출력 설정
ini_set('display_errors', 1);
error_reporting(E_ALL);

header('Content-Type: application/json');

// 사용자 입력 받기
$input = json_decode(file_get_contents("php://input"), true);
$question = trim($input['question'] ?? '');

if (!$question) {
  echo json_encode(['error' => '질문이 비어있습니다.']);
  exit;
}

// ✅ 여기 본인의 Gemini API 키 입력 (https://aistudio.google.com/app/apikey)
$apiKey = 'AIzaSyCOdZQkwRaVXqgVVwJ4lF_XH_PeNd7Nw_Q';  // ← 반드시 본인 키로 교체

$endpoint = "https://generativelanguage.googleapis.com/v1/models/gemini-1.5-pro-002:generateContent?key=$apiKey";

$data = [
  'contents' => [
    [
      'parts' => [
        ['text' => $question]
      ]
    ]
  ]
];

$ch = curl_init($endpoint);
curl_setopt_array($ch, [
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_POST => true,
  CURLOPT_HTTPHEADER => ['Content-Type: application/json'],
  CURLOPT_POSTFIELDS => json_encode($data)
]);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

$result = json_decode($response, true);

// ✅ 성공 응답일 때
if ($httpCode === 200 && isset($result['candidates'][0]['content']['parts'][0]['text'])) {
  echo json_encode([
    'response' => $result['candidates'][0]['content']['parts'][0]['text']
  ]);
} else {
  echo json_encode([
    'error' => $result['error']['message'] ?? 'Gemini 응답 오류',
    'httpCode' => $httpCode,
    'raw' => $response // 필요 시 제거 가능
  ]);
}

```

### `api/check_username.php`
```php
<?php
require_once 'db_connect.php';
header('Content-Type: application/json');

$username = $_GET['username'] ?? '';

if (!$username) {
  echo json_encode(['exists' => false, 'message' => '입력 누락']);
  exit;
}

$stmt = $pdo->prepare("SELECT id FROM users WHERE username = ?");
$stmt->execute([$username]);

$exists = $stmt->fetch() !== false;
echo json_encode(['exists' => $exists]);

```

### `api/db.php`
```php
<?php
$host = 'localhost';
$user = 'root';
$password = '';
$dbname = 'metacritic';

$conn = new mysqli($host, $user, $password, $dbname);

if ($conn->connect_error) {
    die(json_encode(['success' => false, 'message' => 'DB 연결 실패: ' . $conn->connect_error]));
}
?>
```

### `api/delete_game.php`
```php
<?php
require_once 'db_connect.php';
header('Content-Type: application/json');

$data = json_decode(file_get_contents('php://input'), true);
$id = $data['id'] ?? 0;

if (!$id) {
  echo json_encode(['success' => false, 'message' => '게임 ID 누락']);
  exit;
}

$stmt = $pdo->prepare("DELETE FROM games WHERE id = ?");
$success = $stmt->execute([$id]);

echo json_encode(['success' => $success]);
```

### `api/delete_review.php`
```php
<?php
session_start();
require_once '../db_connect.php';
header('Content-Type: application/json');

$input = json_decode(file_get_contents("php://input"), true);
$review_id = $input['id'] ?? null;
$nickname = $_SESSION['nickname'] ?? null;

if (!$review_id || !$nickname) {
  echo json_encode(['success' => false, 'message' => '인증 실패']);
  exit;
}

$stmt = $pdo->prepare("DELETE FROM game_reviews WHERE id = ? AND nickname = ?");
$success = $stmt->execute([$review_id, $nickname]);

echo json_encode(['success' => $success]);

```

### `api/get_game.php`
```php
<?php
require_once(__DIR__ . '/../db_connect.php');
header('Content-Type: application/json');
ini_set('display_errors', 1);
error_reporting(E_ALL);

$id = $_GET['id'] ?? null;
if (!$id) {
  echo json_encode(["error" => "게임 ID 누락"]);
  exit;
}

$stmt = $pdo->prepare("SELECT * FROM games WHERE id = ?");
$stmt->execute([$id]);
$game = $stmt->fetch(PDO::FETCH_ASSOC);

if ($game) {
  echo json_encode($game);
} else {
  echo json_encode(["error" => "게임을 찾을 수 없습니다."]);
}

```

### `api/get_games.php`
```php
<?php
// 에러 표시 설정 (개발환경용)
ini_set('display_errors', 1);
error_reporting(E_ALL);

header('Content-Type: application/json');

// ✅ 정확한 상대경로로 DB 연결
require_once __DIR__ . '/../db_connect.php';

try {
    // 게임 데이터 가져오기
    $stmt = $pdo->prepare("SELECT id, title, description, image_url, meta_score, user_score FROM games");
    $stmt->execute();
    $games = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // ⚠️ JSON 렌더링 불가능한 필드 제거 (선택적)
    /*
    foreach ($games as &$game) {
        foreach ($game as $key => $value) {
            if (is_object($value) || is_array($value)) {
                unset($game[$key]);
            }
        }
    }
    */

    echo json_encode($games);
} catch (Exception $e) {
    // ⛔ JSON 오류 응답
    http_response_code(500);
    echo json_encode([
        'error' => 'DB 오류: ' . $e->getMessage()
    ]);
}

```

### `api/get_reviews.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');

$game_id = $_GET['game_id'] ?? null;

if (!$game_id) {
  echo json_encode(['error' => 'game_id 필요']);
  exit;
}

$stmt = $pdo->prepare("SELECT * FROM game_reviews WHERE game_id = ? ORDER BY created_at DESC");
$stmt->execute([$game_id]);

echo json_encode($stmt->fetchAll(PDO::FETCH_ASSOC));

```

### `api/get_reviews_by_user.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');

$nickname = $_GET['nickname'] ?? '';

if (!$nickname) {
  echo json_encode(['error' => '닉네임이 필요합니다.']);
  exit;
}

$stmt = $pdo->prepare("
  SELECT r.id, r.rating, r.content, r.created_at, g.title AS game_title
  FROM game_reviews r
  JOIN games g ON r.game_id = g.id
  WHERE r.nickname = ?
  ORDER BY r.created_at DESC
");
$stmt->execute([$nickname]);

echo json_encode($stmt->fetchAll(PDO::FETCH_ASSOC));

```

### `api/get_users.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');

$stmt = $pdo->prepare("SELECT id, username, nickname, role FROM users");
$stmt->execute();

$users = $stmt->fetchAll(PDO::FETCH_ASSOC);
echo json_encode($users);

```

### `api/login.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');
session_start();

$input = json_decode(file_get_contents('php://input'), true);
$username = $input['username'] ?? '';
$password = $input['password'] ?? '';

$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);

if ($user && password_verify($password, $user['password'])) {
  $_SESSION['username'] = $user['username'];
  $_SESSION['nickname'] = $user['nickname'];
  $_SESSION['role'] = $user['role'];

  echo json_encode([
    'success' => true,
    'user' => [
      'username' => $user['username'],
      'nickname' => $user['nickname'],
      'role' => $user['role']
    ]
  ]);
} else {
  echo json_encode(['success' => false, 'message' => '아이디 또는 비밀번호가 틀렸습니다.']);
}

```

### `api/logout.php`
```php
<?php
session_start();
session_destroy();
echo json_encode(['success' => true]);
```

### `api/register.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');
session_start();

$input = json_decode(file_get_contents('php://input'), true);
$username = $input['username'] ?? '';
$password = $input['password'] ?? '';
$nickname = $input['nickname'] ?? '';

if (!$username || !$password || !$nickname) {
  echo json_encode(['success' => false, 'message' => '모든 항목을 입력해주세요.']);
  exit;
}

// 중복 체크
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
if ($stmt->fetch()) {
  echo json_encode(['success' => false, 'message' => '이미 존재하는 아이디입니다.']);
  exit;
}

$hashed = password_hash($password, PASSWORD_DEFAULT);

$stmt = $pdo->prepare("INSERT INTO users (username, password, nickname, role) VALUES (?, ?, ?, 'user')");
$success = $stmt->execute([$username, $hashed, $nickname]);

echo json_encode(['success' => $success]);

```

### `api/session_status.php`
```php
<?php
session_start();
header('Content-Type: application/json');

if (isset($_SESSION['username'])) {
  echo json_encode([
    'loggedIn' => true,
    'username' => $_SESSION['username'],
    'nickname' => $_SESSION['nickname'],
    'role' => $_SESSION['role']
  ]);
} else {
  echo json_encode(['loggedIn' => false]);
}

```

### `api/signup_manual.php`
```php
<?php
require_once __DIR__ . '/../db_connect.php';
header('Content-Type: application/json; charset=utf-8');

$data = json_decode(file_get_contents("php://input"), true);

$email = trim($data['email'] ?? '');
$username = trim($data['username'] ?? '');
$password = trim($data['password'] ?? '');

// 유효성 검사
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => '유효한 이메일 형식이 아닙니다']);
    exit;
}

if (strlen($username) < 4 || strlen($username) > 20) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => 'ID는 4~20자여야 합니다']);
    exit;
}

if (strlen($password) < 6) {
    http_response_code(400);
    echo json_encode(['success' => false, 'message' => '비밀번호는 최소 6자 이상이어야 합니다']);
    exit;
}

// 중복 체크
$stmt = $pdo->prepare("SELECT id FROM users WHERE email = ?");
$stmt->execute([$email]);
if ($stmt->fetch()) {
    http_response_code(409);
    echo json_encode(['success' => false, 'message' => '이미 등록된 이메일입니다']);
    exit;
}

$stmt = $pdo->prepare("SELECT id FROM users WHERE username = ?");
$stmt->execute([$username]);
if ($stmt->fetch()) {
    http_response_code(409);
    echo json_encode(['success' => false, 'message' => '이미 사용 중인 ID입니다']);
    exit;
}

// 비밀번호 해싱
$hashed = password_hash($password, PASSWORD_DEFAULT);

// 저장
$stmt = $pdo->prepare("INSERT INTO users (email, username, password, signup_method, role) VALUES (?, ?, ?, 'manual', 'user')");
$success = $stmt->execute([$email, $username, $hashed]);

echo json_encode(['success' => $success]);
?>

```

### `api/update_games.php`
```php
<?php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

require_once __DIR__ . '/../db_connect.php';

$apiKey = '672726e2c0b245d99db2592da35bfdc1'; // 🔑 여기에 본인의 RAWG API 키를 입력하세요
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

### `api/update_user_role.php`
```php
<?php
require_once '../db_connect.php';
header('Content-Type: application/json');
ini_set('display_errors', 1);
error_reporting(E_ALL);

$input = json_decode(file_get_contents("php://input"), true);

$id = isset($input['id']) ? (int)$input['id'] : null;
$role = isset($input['role']) ? trim($input['role']) : null;

// 정확하게 유효성 체크
if ($id === null || $id <= 0 || !$role || !in_array($role, ['user', 'admin'])) {
  echo json_encode([
    'success' => false,
    'message' => "잘못된 요청입니다",
    'debug' => ['id' => $id, 'role' => $role]
  ]);
  exit;
}

try {
  $stmt = $pdo->prepare("UPDATE users SET role = ? WHERE id = ?");
  $success = $stmt->execute([$role, $id]);

  echo json_encode([
    'success' => $success,
    'message' => $success ? '권한 변경 성공' : '쿼리 실패',
    'debug' => ['id' => $id, 'role' => $role]
  ]);
} catch (PDOException $e) {
  echo json_encode([
    'success' => false,
    'message' => '쿼리 오류: ' . $e->getMessage()
  ]);
}

```

## 🗄️ 데이터베이스 스키마 (`db/schema.sql`)

### --- games 테이블
```sql
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

### --- game_reviews 테이블
```sql
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

### --- users 테이블 
```sql
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

### -- -tasks.json
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
