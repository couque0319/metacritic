# 📦 Frontend - reactproject

React 기반 게임 리뷰 프로젝트의 **프론트엔드**입니다.

## 📁 디렉토리 구조

```
frontend/
├── copy-to-xampp.js
├── package.json
├── package-lock.json
├── .env
├── .gitignore
├── README.md
├── build/
│   ├── index.html
│   ├── favicon.ico
│   ├── manifest.json
│   ├── robots.txt
│   └── static/
│       ├── css/
│       └── js/
├── node_modules/
```

## 📌 주요 설명

- **`package.json`** : 프로젝트 의존성과 실행 스크립트 정의
- **`build/`** : `npm run build`로 생성된 정적 리소스
- **`.env`** : 환경 변수 정의 파일
- **`copy-to-xampp.js`** : 빌드 결과물을 XAMPP로 복사하는 유틸

## 🧩 JSX 파일 코드

### `src/components/AIBar/AIBar.jsx`
```jsx
import React, { useState } from 'react';
import './AIBar.css';

function AIBar() {
  const [input, setInput] = useState('');
  const [response, setResponse] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!input.trim()) return;

    setLoading(true);
    setResponse('');

    try {
      const res = await fetch('http://localhost/reactproject/api/ask_ai.php', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ question: input })
      });

      const data = await res.json();
      console.log("Gemini 응답:", data);

      if (data.response) {
        setResponse(data.response);
      } else if (data.error) {
        setResponse('❗ Gemini 오류: ' + data.error);
      } else {
        setResponse('AI 응답 오류');
      }
    } catch (err) {
      setResponse('요청 실패: ' + err.message);
    }

    setLoading(false);
    setInput('');
  };

  return (
    <div className={`ai-container ${response ? 'expanded' : ''}`}>
      <form className="ai-bar" onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="AI에게 물어보세요..."
          value={input}
          onChange={(e) => setInput(e.target.value)}
        />
        <button type="submit" disabled={loading}>
          {loading ? '...' : '▶'}
        </button>
      </form>

      {response && (
        <div className="ai-response">
          <strong>AI 응답:</strong>
          <p>{response}</p>
        </div>
      )}
    </div>
  );
}

export default AIBar;

```

### `src/components/Auth/AuthModal.jsx`
```jsx
// src/components/Auth/AuthModal.jsx
import React, { useState } from 'react';
import LoginForm from './LoginForm';
import SignupForm from './SignupForm';
import './AuthModal.css';

function AuthModal({ onClose }) {
  const [mode, setMode] = useState('login');

  return (
    <div className="auth-modal-overlay">
      <div className="auth-modal">
        <button className="close-button" onClick={onClose}>×</button>
        {mode === 'login' ? (
          <>
            <h2>로그인</h2>
            <LoginForm onClose={onClose} />
            <p onClick={() => setMode('signup')}>계정이 없으신가요? <strong>회원가입</strong></p>
          </>
        ) : (
          <>
            <h2>회원가입</h2>
            <SignupForm onClose={onClose} />
            <p onClick={() => setMode('login')}>이미 계정이 있으신가요? <strong>로그인</strong></p>
          </>
        )}
      </div>
    </div>
  );
}

export default AuthModal;

```

### `src/components/Auth/LoginForm.jsx`
```jsx
import React, { useState } from 'react';

function LoginForm({ onClose }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  const handleLogin = async () => {
    if (!username || !password) {
      alert('아이디와 비밀번호를 입력해주세요.');
      return;
    }

    setLoading(true);
    try {
      const res = await fetch('/reactproject/api/login.php', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });

      const data = await res.json();
      if (data.success) {
       alert('로그인 성공');
        onClose();
        setTimeout(() => {
        window.location.reload();
    }, 100); // 짧은 지연으로 모달 정상 닫힘

      } else {
        alert(data.message || '로그인 실패');
      }
    } catch (err) {
      alert('서버 오류가 발생했습니다.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-form">
      <input
        type="text"
        placeholder="아이디"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        className="auth-input"
      />
      <input
        type="password"
        placeholder="비밀번호"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        className="auth-input"
      />
      <button onClick={handleLogin} className="auth-button" disabled={loading}>
        {loading ? '처리 중...' : '로그인'}
      </button>
    </div>
  );
}

export default LoginForm;

```

### `src/components/Auth/SignupForm.jsx`
```jsx
// SignupForm.jsx
import React, { useState } from 'react';

function SignupForm({ onClose }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [nickname, setNickname] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSignup = async () => {
    if (!username || !password || !nickname) {
      alert('모든 항목을 입력해주세요.');
      return;
    }

    setLoading(true);
    try {
      const res = await fetch('/reactproject/api/register.php', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password, nickname })
      });

      const data = await res.json();
      if (data.success) {
        alert('✅ 회원가입 완료! 로그인해주세요.');
        onClose();
      } else {
        alert(data.message || '❌ 회원가입 실패');
      }
    } catch (error) {
      alert('서버 오류가 발생했습니다.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="auth-form">
      <input
        type="text"
        placeholder="아이디"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        className="auth-input"
      />
      <input
        type="text"
        placeholder="닉네임"
        value={nickname}
        onChange={(e) => setNickname(e.target.value)}
        className="auth-input"
      />
      <input
        type="password"
        placeholder="비밀번호"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        className="auth-input"
      />
      <button onClick={handleSignup} className="auth-button" disabled={loading}>
        {loading ? '처리 중...' : '회원가입'}
      </button>
    </div>
  );
}

export default SignupForm;

```

### `src/components/GameCard/GameCard.jsx`
```jsx
// GameCard.jsx
import React from 'react';
import './GameCard.css';
import { Link } from 'react-router-dom';

function getScoreClass(score) {
  if (score >= 75) return 'high';
  if (score >= 50) return 'mid';
  return 'low';
}

function GameCard({ game }) {
  // 안전하게 데이터 fallback 설정
  const title = typeof game.title === 'string' ? game.title : '제목 없음';
  const description = typeof game.description === 'string' ? game.description.slice(0, 100) + '...' : '설명이 없습니다.';
  const metaScore = typeof game.meta_score === 'number' ? game.meta_score : 'N/A';
  const userScore = typeof game.user_score === 'number' ? game.user_score : 'N/A';

  return (
    <div className="game-card">
      <Link to={`/games/${game.id}`} className="game-link">
        {game.image_url ? (
          <img
            src={game.image_url}
            alt={title}
            className="game-image"
            onError={(e) => (e.target.style.display = 'none')}
          />
        ) : (
          <div style={{ height: '200px', background: '#444' }}>이미지 없음</div>
        )}
        <h3 className="game-title">{title}</h3>
      </Link>

      <p className="description">{description}</p>
      <p className="meta-score" data-score={getScoreClass(game.meta_score)}>
        메타 점수: {metaScore}
      </p>
      <p className="user-score">
        유저 점수: {userScore}
      </p>
    </div>
  );
}

export default GameCard;

```

### `src/components/Header/Header.jsx`
```jsx
import React, { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import './Header.css';

function Header({ onLoginClick }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch('/reactproject/api/session_status.php', {
      credentials: 'include'
    })
      .then(res => res.json())
      .then(data => {
      console.log("session_status 응답:", data);
      if (data.loggedIn) {
       // user 관련 정보만 따로 저장
       setUser({ username: data.username, role: data.role });
     }
    });
  }, []);

  const handleLogout = () => {
    fetch('/reactproject/api/logout.php', {
      credentials: 'include'
    }).then(() => {
      setUser(null);
      window.location.reload();
    });
  };

  return (
    <header className="header">
      <div className="header-inner">
        <div className="logo">
          <Link to="/">
            <span role="img" aria-label="controller">🎮</span> <strong>GameReview Hub</strong>
          </Link>
        </div>

        <nav className="nav-menu">
          <Link to="/">Home</Link>
          <Link to="/games">Games</Link>
          {user?.role === 'admin' && <Link to="/admin">Admin</Link>}

          {user ? (
            <>
              <span className="username">👤 {user.nickname}</span>
              <Link to="/profile">
                <button className="header-button">Profile</button>
              </Link>
              <button className="header-button" onClick={handleLogout}>Logout</button>
            </>
          ) : (
            <button className="header-button" onClick={onLoginClick}>Register</button>
          )}
        </nav>
      </div>
    </header>
  );
}

export default Header;

```

### `src/components/Review/ReviewForm.jsx`
```jsx
// ReviewForm.jsx
import React, { useEffect, useState } from 'react';
import './ReviewForm.css';

function ReviewForm({ gameId, onReviewAdded }) {
  const [nickname, setNickname] = useState('');
  const [rating, setRating] = useState('');
  const [content, setContent] = useState('');
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  useEffect(() => {
    fetch('/reactproject/api/session_status.php', {
      credentials: 'include'
    })
      .then(res => res.json())
      .then(data => {
        setIsLoggedIn(data.loggedIn);
        if (data.loggedIn) {
          setNickname(data.nickname);  // ✅ 정확히 nickname 사용
        }
      });
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!isLoggedIn) {
      alert("로그인 후 이용이 가능합니다.");
      window.dispatchEvent(new Event('open-auth-modal'));
      return;
    }

    const res = await fetch('/reactproject/api/add_review.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ game_id: gameId, nickname, rating, content })
    });

    const result = await res.json();
    if (result.success) {
      setRating('');
      setContent('');
      onReviewAdded();  // 리뷰 목록 갱신
    } else {
      alert('리뷰 등록에 실패했습니다.');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="review-form">
      <input type="text" placeholder="닉네임" value={nickname} disabled />
      <input
        type="number"
        placeholder="평점 (0~10)"
        value={rating}
        onChange={(e) => setRating(e.target.value)}
        required
        min="0"
        max="10"
      />
      <textarea
        placeholder="리뷰를 작성하세요"
        value={content}
        onChange={(e) => setContent(e.target.value)}
        required
      />
      <button type="submit">리뷰 등록</button>
    </form>
  );
}

export default ReviewForm;

```

### `src/components/Review/ReviewList.jsx`
```jsx
// ReviewList.jsx
import React, { useEffect, useState } from 'react';
import './ReviewList.css';

function ReviewList({ gameId }) {
  const [reviews, setReviews] = useState([]);
  const [currentUser, setCurrentUser] = useState('');

  useEffect(() => {
    fetch(`/reactproject/api/get_reviews.php?game_id=${gameId}`)
      .then(res => res.json())
      .then(setReviews);

    fetch('/reactproject/api/session_status.php', {
      credentials: 'include'
    })
      .then(res => res.json())
      .then(data => {
        if (data.loggedIn) {
          setCurrentUser(data.nickname);
        }
      });
  }, [gameId]);

  const handleDelete = (id) => {
    if (!window.confirm('정말 삭제하시겠습니까?')) return;

    fetch('/reactproject/api/delete_review.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ id })
    })
      .then(res => res.json())
      .then(result => {
        if (result.success) {
          setReviews(reviews.filter(r => r.id !== id));
        } else {
          alert('삭제에 실패했습니다.');
        }
      });
  };

  if (reviews.length === 0) {
    return <p className="no-review">작성된 리뷰가 없습니다.</p>;
  }

  return (
    <div className="review-list">
      {reviews.map(review => (
        <div key={review.id} className="review-card">
          <div className={`review-score score-${Math.floor(review.rating)}`}>
            {review.rating}
          </div>
          <div className="review-info">
            <div className="review-meta">
              <strong>{review.nickname}</strong>
              <small>{review.created_at}</small>
            </div>
            <p className="review-content">{review.content}</p>
            {currentUser === review.nickname && (
              <div className="review-actions">
                <button onClick={() => handleDelete(review.id)}>삭제</button>
              </div>
            )}
          </div>
        </div>
      ))}
    </div>
  );
}

export default ReviewList;

```

### `src/pages/Admin/Admin.jsx`
```jsx
// src/pages/Admin.jsx
import React, { useEffect, useState } from 'react';

function Admin() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('http://localhost/reactproject/api/get_users.php')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);

  const updateRole = (username, newRole) => {
    fetch('http://localhost/reactproject/api/update_user_role.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, role: newRole })
    })
    .then(res => res.json())
    .then(data => alert(data.message));
  };

  const deleteReview = (reviewId) => {
    fetch('http://localhost/reactproject/api/delete_review.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ reviewId })
    })
    .then(res => res.json())
    .then(data => alert(data.message));
  };

  return (
    <div>
      <h2>회원 관리</h2>
      <table>
        <thead>
          <tr>
            <th>아이디</th>
            <th>권한</th>
            <th>권한 변경</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.username}>
              <td>{user.username}</td>
              <td>{user.role}</td>
              <td>
                <button onClick={() => updateRole(user.username, 'admin')}>Admin</button>
                <button onClick={() => updateRole(user.username, 'user')}>User</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default Admin;
```

### `src/pages/Admin/AdminPanel.jsx`
```jsx
import React, { useEffect, useState } from 'react';
import './AdminPanel.css';

function AdminPanel() {
  const [users, setUsers] = useState([]);
  const [selectedNickname, setSelectedNickname] = useState(null);
  const [userReviews, setUserReviews] = useState([]);

  useEffect(() => {
    fetch('/reactproject/api/get_users.php')
      .then(res => res.json())
      .then(setUsers);
  }, []);

  const changeRole = (userId, newRole) => {
    fetch('/reactproject/api/update_user_role.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id: userId, role: newRole })
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert('권한 변경 완료');
          // ✅ 상태에서 해당 유저의 role만 업데이트
          setUsers(prev =>
            prev.map(u => (u.id === userId ? { ...u, role: newRole } : u))
          );
        } else {
          alert('변경 실패');
          console.log(data.message);
        }
      });
  };

  const handleUserClick = (nickname) => {
    setSelectedNickname(nickname);
    fetch(`/reactproject/api/get_reviews_by_user.php?nickname=${nickname}`)
      .then(res => res.json())
      .then(setUserReviews);
  };

  const deleteReview = (reviewId) => {
    if (!window.confirm('정말로 이 리뷰를 삭제하시겠습니까?')) return;

    fetch('/reactproject/api/delete_review.php', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id: reviewId })
    })
      .then(res => res.json())
      .then(data => {
        if (data.success) {
          alert('리뷰 삭제 완료');
          setUserReviews(prev => prev.filter(r => r.id !== reviewId));
        } else {
          alert('리뷰 삭제 실패');
        }
      });
  };

  return (
    <div className="admin-container">
      <h2>👑 관리자 패널</h2>

      <table className="user-table">
        <thead>
          <tr>
            <th>아이디</th>
            <th>닉네임</th>
            <th>권한</th>
            <th>권한 변경</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.username}</td>
              <td>
                <button
                  className="nickname-button"
                  onClick={() => handleUserClick(user.nickname)}
                >
                  {user.nickname}
                </button>
              </td>
              <td>{user.role}</td> {/* ✅ 현재 권한 표시 */}
              <td>
                <button
                  className="role-user"
                  onClick={() => changeRole(user.id, 'user')}
                >
                  user
                </button>
                <button
                  className="role-admin"
                  onClick={() => changeRole(user.id, 'admin')}
                >
                  admin
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      {selectedNickname && (
        <>
          <h3>📝 <span style={{ color: '#0ff' }}>{selectedNickname}</span> 님의 리뷰 목록</h3>
          {userReviews.length === 0 ? (
            <p>작성한 리뷰가 없습니다.</p>
          ) : (
            <div className="review-list">
              {userReviews.map(review => (
                <div className="review-card" key={review.id}>
                  <div className="review-card-header">
                    <div>
                      <p><strong>게임:</strong> {review.game_title}</p>
                      <p><strong>평점:</strong> {review.rating}</p>
                    </div>
                    <button
                      className="review-delete-btn"
                      onClick={() => deleteReview(review.id)}
                    >
                      리뷰 삭제
                    </button>
                  </div>
                  <p className="review-card-content">{review.content}</p>
                </div>
              ))}
            </div>
          )}
        </>
      )}
    </div>
  );
}

export default AdminPanel;

```

### `src/pages/GameDetail/GameDetail.jsx`
```jsx
// src/pages/GameDetail/GameDetail.jsx
import React, { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import ReviewList from '../../components/Review/ReviewList';
import ReviewForm from '../../components/Review/ReviewForm';
import './GameDetail.css';

function GameDetail() {
  const { id } = useParams();
  const [game, setGame] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(`/reactproject/api/get_game.php?id=${id}`)
      .then(res => {
        if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
        return res.json();
      })
      .then(data => {
        if (data?.error) {
          setError(data.error);
        } else {
          setGame(data);
        }
      })
      .catch(err => {
        console.error('게임 상세 정보 불러오기 실패:', err);
        setError('서버 오류가 발생했습니다.');
      });
  }, [id]);

  if (error) return <p className="game-detail-wrapper">⚠️ {error}</p>;
  if (!game) return <p className="game-detail-wrapper">로딩 중...</p>;

  return (
    <div className="game-detail-wrapper">
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
    </div>
  );
}

export default GameDetail;

```

### `src/pages/GameList/GameList.jsx`
```jsx
import React, { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import './GameList.css';

function GameList() {
  const [games, setGames] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');

  useEffect(() => {
    fetch('/reactproject/api/get_games.php')
  .then(res => {
    if (!res.ok) throw new Error(`HTTP error: ${res.status}`);
    return res.json();
  })
  .then(data => setGames(data))
  .catch(err => console.error("게임 데이터를 불러오는 중 오류 발생:", err));

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
          <Link to={`/games/${game.id}`} key={game.id} className="horizontal-game-card-link">
            <div className="horizontal-game-card">
              <img src={game.image_url} alt={game.title} className="horizontal-game-image" />
              <div className="horizontal-game-info">
                <h3 className="horizontal-game-title">{game.title}</h3>
                <p className="horizontal-description">{game.description?.slice(0, 120)}...</p>
                <p className="meta-score">메타 점수: {game.meta_score}</p>
                <p className="user-score">유저 점수: {game.user_score}</p>
              </div>
            </div>
          </Link>
        ))}
      </div>
    </main>
  );
}

export default GameList;

```

### `src/pages/Home/Home.jsx`
```jsx
// src/pages/Home/Home.jsx
import React, { useEffect, useState } from 'react';
import GameCard from '../../components/GameCard/GameCard';
import './Home.css';

function Home() {
  const [games, setGames] = useState([]);

  useEffect(() => {
    fetch('/reactproject/api/get_games.php')
      .then(res => res.json())
      .then(data => {
        console.log("게임 데이터 개수:", data.length);
        console.log("샘플 게임 구조:", data[0]);
        setGames(data);
      })
      .catch(err => console.error("게임 데이터를 불러오는 중 오류:", err));
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

### `src/pages/Profile/Profile.jsx`
```jsx
// Profile.jsx
import React, { useEffect, useState } from 'react';
import './Profile.css';
import { Link } from 'react-router-dom';

function Profile() {
  const [user, setUser] = useState(null);
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    fetch('/reactproject/api/session_status.php', {
      credentials: 'include'
    })
      .then(res => res.json())
      .then(data => {
        if (data.loggedIn) {
          setUser(data);

          // 사용자 리뷰 불러오기
          fetch(`/reactproject/api/get_reviews_by_user.php?nickname=${data.nickname}`)
            .then(res => res.json())
            .then(setReviews);
        }
      });
  }, []);

  if (!user) return <p>로그인이 필요합니다.</p>;

  return (
    <div className="profile-wrapper">
      <aside className="profile-sidebar">
        <h2>My Profile</h2>
        <nav>
          <ul>
            <li className="active">MY RATINGS & REVIEWS</li>
            <li><Link to="#">MY ACCOUNT</Link></li>
            <li><Link to="#" onClick={() => {
              fetch('/reactproject/api/logout.php', {
                credentials: 'include'
              }).then(() => window.location.href = '/');
            }}>SIGN OUT</Link></li>
          </ul>
        </nav>
      </aside>

      <main className="profile-main">
        <h3>My Ratings & Reviews</h3>
        {reviews.length === 0 ? (
          <p className="no-review">You haven’t rated anything yet</p>
        ) : (
          <ul className="review-list">
            {reviews.map(r => (
              <li key={r.id}>
                <Link to={`/games/${r.game_id}`}>
                  <strong>{r.game_title}</strong> - ⭐ {r.rating}
                </Link>
                <p>{r.content}</p>
              </li>
            ))}
          </ul>
        )}
      </main>
    </div>
  );
}

export default Profile;

```
