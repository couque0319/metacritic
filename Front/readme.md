# 🎮 GameReview Hub - React Application

This repository contains the source code for GameReview Hub, a React application for browsing and reviewing games.

## 📋 Table of Contents
- [Core Files](#-core-files)
- [Components](#-components)
- [Pages](#-pages)

## 🧩 Core Files

### App.js
```jsx
import React from 'react';
import { Routes, Route } from 'react-router-dom';

import Home from './pages/Home/Home';
import GameList from './pages/GameList/GameList';
import GameDetail from './pages/GameDetail/GameDetail';
import Login from './pages/Login/Login';
import Signup from './pages/Signup/Signup';

import Header from './components/Header/Header';

const App = () => {
  return (
    <>
      <Header />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/games" element={<GameList />} />
        <Route path="/games/:id" element={<GameDetail />} />
        <Route path="/login" element={<Login />} />
        <Route path="/signup" element={<Signup />} />
        <Route path="*" element={<div>페이지를 찾을 수 없습니다.</div>} />
      </Routes>
    </>
  );
};

export default App;
```

### index.js
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { BrowserRouter } from 'react-router-dom';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <BrowserRouter basename='/reactproject'>		
    <App />
  </BrowserRouter>
);
```

### App.css
```css
body {
  margin: 0;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  background-color: #1b1b1b;
  color: #eee;
}

a {
  color: #4fc3f7;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}

.card {
  background-color: #2a2a2a;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
  box-shadow: 0 0 5px rgba(0,0,0,0.5);
}

.score-badge {
  padding: 5px 10px;
  border-radius: 5px;
  color: white;
  font-weight: bold;
}

.score-high {
  background-color: #66bb6a;
}

.score-medium {
  background-color: #fdd835;
}

.score-low {
  background-color: #ef5350;
}
```

## 🎨 Components

### Header Component

#### Header.jsx
```jsx
import React from 'react';
import { Link } from 'react-router-dom';
import './Header.css';

function Header() {
  return (
    <header className="site-header">
      <div className="logo">🎮 GameReview Hub</div>
      <nav className="nav-links">
        <Link to="/">Home</Link>
        <Link to="/games">Games</Link>
        <button className="register-button">Register</button>
      </nav>
    </header>
  );
}

export default Header;
```

#### Header.css
```css
.site-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  background-color: #000;
  color: white;
}
.logo { font-size: 1.5rem; font-weight: bold; }
.nav-links { display: flex; align-items: center; gap: 1rem; }
.nav-links a { color: #ffc107; text-decoration: none; font-weight: 500; }
.register-button {
  background-color: #ffc107;
  color: black;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
}
.register-button:hover { background-color: #e0a800; }
```

### GameCard Component

#### GameCard.jsx
```jsx
import React from 'react';
import './GameCard.css';
import { Link } from 'react-router-dom';

function getScoreClass(score) {
  if (score >= 75) return 'high';
  if (score >= 50) return 'mid';
  return 'low';
}

function GameCard({ game }) {
  return (
    <div className="game-card">
      <Link to={`/games/${game.id}`}>
        <img src={game.image_url} alt={game.title} className="game-image" />
        <h3 className="game-title">{game.title}</h3>
      </Link>
      <p className="description">
        {game.description ? game.description.slice(0, 100) + '...' : ''}
      </p>
      <p className="meta-score" data-score={getScoreClass(game.meta_score)}>
        메타 점수: {game.meta_score}
      </p>
      <p className="user-score">유저 점수: {game.user_score}</p>
    </div>
  );
}

export default GameCard;
```

#### GameCard.css
```css
.game-card {
  background-color: #2a2a2a;
  padding: 1rem;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  text-align: center;
  color: #f1f1f1;
}
.game-link { text-decoration: none; color: inherit; }
.game-image { width: 100%; height: auto; border-radius: 6px; }
.game-title {
  font-size: 1.1rem;
  margin: 0.7rem 0 0.5rem;
  color: #ffffff;
  font-weight: bold;
  border-bottom: 1px solid #555;
  padding-bottom: 0.3rem;
}
.description {
  font-size: 0.95rem;
  color: #f5f5f5;
  margin-bottom: 0.7rem;
}
.meta-score {
  background-color: #4caf50;
  color: white;
  display: inline-block;
  padding: 0.2rem 0.6rem;
  border-radius: 5px;
  font-weight: bold;
  font-size: 0.9rem;
}
.user-score {
  font-size: 0.9rem;
  margin-top: 0.4rem;
  color: #ffffff;
}
```

### ReviewForm Component

#### ReviewForm.jsx
```jsx
import React, { useState } from 'react';

function ReviewForm({ gameId }) {
  const [nickname, setNickname] = useState('');
  const [rating, setRating] = useState('');
  const [content, setContent] = useState('');

  const handleSubmit = e => {
    e.preventDefault();
    fetch('http://localhost/server/api/add_review.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ game_id: gameId, nickname, rating, content })
    })
      .then(response => response.text())
      .then(() => {
        setNickname('');
        setRating('');
        setContent('');
      });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="닉네임" value={nickname} onChange={e => setNickname(e.target.value)} required />
      <input type="number" placeholder="평점" value={rating} onChange={e => setRating(e.target.value)} required min="0" max="10" />
      <textarea placeholder="리뷰를 작성하세요" value={content} onChange={e => setContent(e.target.value)} required></textarea>
      <button type="submit">리뷰 등록</button>
    </form>
  );
}

export default ReviewForm;
```

### ReviewList Component

#### ReviewList.jsx
```jsx
import React, { useEffect, useState } from 'react';

function ReviewList({ gameId }) {
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    fetch(`http://localhost/server/api/get_reviews.php?game_id=${gameId}`)
      .then(response => response.json())
      .then(data => setReviews(data));
  }, [gameId]);

  return (
    <div>
      <h3>리뷰 목록</h3>
      {reviews.map(review => (
        <div key={review.id}>
          <p><strong>{review.nickname}</strong> - 평점: {review.rating}</p>
          <p>{review.content}</p>
        </div>
      ))}
    </div>
  );
}

export default ReviewList;
```

## 📄 Pages

### Home Page

#### Home.jsx
```jsx
import React, { useEffect, useState } from 'react';
import GameCard from '../../components/GameCard/GameCard';
import './Home.css';

function Home() {
  const [games, setGames] = useState([]);

  useEffect(() => {
    fetch('/server/api/get_games.php')
      .then(res => res.json())
      .then(data => setGames(data));
  }, []);

  return (
    <main className="home">
      <h2>최신 게임</h2>
      <div className="game-list">
        {games.map(game => (
          <GameCard key={game.id} game={game} />
        ))}
      </div>
    </main>
  );
}

export default Home;
```

#### Home.css
```css
.home {
  padding: 2rem;
}

.game-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
}
```

### Game List Page

#### GameList.jsx
```jsx
import React, { useEffect, useState } from 'react';
import './GameList.css';

function GameList() {
  const [games, setGames] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');

  useEffect(() => {
    fetch('/server/api/get_games.php')
      .then(res => res.json())
      .then(data => setGames(data));
  }, []);

  const filteredGames = games.filter(game =>
    game.title.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <main className="game-list-wrapper">
      <h2 className="page-title">전체 게임 목록</h2>
      <input
        type="text"
        className="search-input"
        placeholder="게임 제목 검색..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
      <div className="game-horizontal-list">
        {filteredGames.map(game => (
          <div className="horizontal-game-card" key={game.id}>
            <img src={game.image_url} alt={game.title} className="horizontal-game-image" />
            <div className="horizontal-game-info">
              <h3 className="horizontal-game-title">{game.title}</h3>
              <p className="horizontal-description">{game.description?.slice(0, 120)}...</p>
              <p className="meta-score">메타 점수: {game.meta_score}</p>
              <p className="user-score">유저 점수: {game.user_score}</p>
            </div>
          </div>
        ))}
      </div>
    </main>
  );
}

export default GameList;
```

#### GameList.css
```css
body {
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #1c1c1c;
  color: #f0f0f0;
}

.site-title {
  font-size: 4rem;
  color: #ffffff;
  margin-bottom: 0.3rem;
  font-weight: 700;
}

.nav-links {
  font-size: 2rem;
  margin-bottom: 1rem;
}

.nav-links a {
  color: #f0f0f0;
  margin-right: 0.8rem;
  text-decoration: none;
  font-weight: 500;
}

.nav-links a:hover {
  color: #f5c518;
}

.game-list-wrapper {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 2rem;
}

.page-title {
  font-size: 1.8rem;
  color: #f5c518;
  border-bottom: 1px solid #444;
  padding-bottom: 0.5rem;
  margin-bottom: 1rem;
}

.search-input {
  width: 100%;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  border-radius: 6px;
  border: none;
  margin-bottom: 1.5rem;
  background-color: #2e2e2e;
  color: #fff;
}

.search-input::placeholder {
  color: #aaa;
}

.game-horizontal-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.horizontal-game-card {
  display: flex;
  gap: 1.2rem;
  padding: 1rem;
  border-radius: 10px;
  background-color: #2b2b2b;
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.4);
  align-items: center;
}

.horizontal-game-image {
  width: 320px;
  height: 180px;
  object-fit: cover;
  border-radius: 8px;
}

.horizontal-game-info {
  flex: 1;
}

.horizontal-game-title {
  font-size: 1.6rem;
  color: #ffffff;
  font-weight: 800;
  margin-bottom: 0.5rem;
}

.horizontal-description {
  font-size: 1.05rem;
  color: #f2f2f2;
  margin-bottom: 0.6rem;
  line-height: 1.6;
}

.meta-score {
  font-size: 1rem;
  font-weight: 600;
  background-color: #66bb6a;
  display: inline-block;
  padding: 4px 8px;
  border-radius: 4px;
  color: #000;
  margin-right: 0.5rem;
}

.user-score {
  font-size: 1rem;
  color: #dddddd;
}
```

### Game Detail Page

#### GameDetail.jsx
```jsx
import React, { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import ReviewList from '../../components/ReviewList/ReviewList';
import ReviewForm from '../../components/ReviewForm/ReviewForm';
import './GameDetail.css';

function GameDetail() {
  const { id } = useParams();
  const [game, setGame] = useState(null);

  useEffect(() => {
    fetch(`/server/api/get_game.php?id=${id}`)
      .then((res) => res.json())
      .then(setGame);
  }, [id]);

  if (!game) return <p>로딩 중...</p>;

  return (
    <div className="game-detail">
      <h2>{game.title}</h2>
      <img src={game.image_url} alt={game.title} className="game-detail-image" />
      <p className="description">{game.description}</p>
      <p className="meta-score">메타 점수: {game.meta_score}</p>
      <p className="user-score">유저 점수: {game.user_score}</p>

      <h3>리뷰</h3>
      <ReviewList gameId={id} />
      <ReviewForm gameId={id} />
    </div>
  );
}

export default GameDetail;
```

#### GameDetail.css
```css
.game-detail {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  font-family: 'Helvetica Neue', sans-serif;
}

.game-detail h2 {
  font-size: 2rem;
  margin-bottom: 1rem;
}

.game-detail-image {
  width: 100%;
  max-height: 400px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 1rem;
}

.description {
  font-size: 1rem;
  color: #333;
  margin-bottom: 1rem;
  line-height: 1.6;
}

.meta-score,
.user-score {
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: #444;
}

h3 {
  margin-top: 2rem;
  font-size: 1.5rem;
  border-bottom: 1px solid #ccc;
  padding-bottom: 0.5rem;
}
```

### Login Page

#### Login.jsx
```jsx
import React, { useState } from 'react';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = () => {
    fetch('/server/api/login.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert('로그인 성공');
          localStorage.setItem('user', JSON.stringify(data.user));
          window.location.href = '/';
        } else {
          alert(data.message);
        }
      });
  };

  return (
    <div className="login-container">
      <h2>로그인</h2>
      <input
        type="email"
        placeholder="이메일"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      <input
        type="password"
        placeholder="비밀번호"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      <button onClick={handleLogin}>로그인</button>
    </div>
  );
}

export default Login;
```

### Signup Page

#### Signup.jsx
```jsx
import React, { useState } from 'react';

function Signup() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [mode, setMode] = useState('manual');

  const handleManualSignup = () => {
    fetch('/server/api/signup_manual.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert('가입 완료');
        } else {
          alert(data.message);
        }
      });
  };

  const handleEmailSignup = () => {
    fetch('/reactproject/api/signup_email.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert('이메일 인증 완료');
        } else {
          alert(data.message);
        }
      });
  };

  return (
    <div className="signup-container">
      <h2>회원가입</h2>
      <div>
        <button onClick={() => setMode('manual')}>ID/비밀번호로 가입</button>
        <button onClick={() => setMode('email')}>이메일로 가입</button>
      </div>

      {mode === 'manual' && (
        <div>
          <input
            type="email"
            placeholder="이메일"
            value={email}
            onChange={e => setEmail(e.target.value)}
          />
          <input
            type="password"
            placeholder="비밀번호"
            value={password}
            onChange={e => setPassword(e.target.value)}
          />
          <button onClick={handleManualSignup}>가입하기</button>
        </div>
      )}

      {mode === 'email' && (
        <div>
          <input
            type="email"
            placeholder="이메일"
            value={email}
            onChange={e => setEmail(e.target.value)}
          />
          <button onClick={handleEmailSignup}>인증 메일 보내기</button>
        </div>
      )}
    </div>
  );
}

export default Signup;
```
