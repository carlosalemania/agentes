# OAuth/JWT Security Expert - Examples

---

## Example: Express API with JWT Authentication

```javascript
// Complete Express API with JWT auth

const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

// Login endpoint
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  // Find user
  const user = await db.users.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Verify password
  const validPassword = await bcrypt.compare(password, user.passwordHash);
  if (!validPassword) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }

  // Generate tokens
  const accessToken = jwt.sign(
    { sub: user.id, email: user.email },
    JWT_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { sub: user.id },
    JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  // Store refresh token
  await db.refreshTokens.insert({
    userId: user.id,
    token: refreshToken,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  });

  res.json({
    accessToken,
    refreshToken,
    expiresIn: 900  // 15 minutes
  });
});

// Refresh endpoint
app.post('/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }

  try {
    const decoded = jwt.verify(refreshToken, JWT_REFRESH_SECRET);

    // Check if token exists in database
    const storedToken = await db.refreshTokens.findOne({
      userId: decoded.sub,
      token: refreshToken
    });

    if (!storedToken) {
      return res.status(401).json({ error: 'Invalid refresh token' });
    }

    // Generate new access token
    const accessToken = jwt.sign(
      { sub: decoded.sub },
      JWT_SECRET,
      { expiresIn: '15m' }
    );

    res.json({
      accessToken,
      expiresIn: 900
    });
  } catch (error) {
    return res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// Protected route
app.get('/api/profile', authenticateJWT, async (req, res) => {
  const user = await db.users.findById(req.user.sub);
  res.json(user);
});

function authenticateJWT(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

app.listen(3000);
```

---

## Example: React OAuth 2.0 PKCE Client

```javascript
// React OAuth client with PKCE

import { useState, useEffect } from 'react';

function App() {
  const [user, setUser] = useState(null);

  const oauth = new OAuth2Client({
    clientId: 'my-client-id',
    authorizationEndpoint: 'https://auth.example.com/oauth/authorize',
    tokenEndpoint: 'https://auth.example.com/oauth/token',
    redirectUri: 'http://localhost:3000/callback'
  });

  useEffect(() => {
    // Handle OAuth callback
    if (window.location.pathname === '/callback') {
      oauth.handleCallback().then(() => {
        window.location.href = '/';
      });
    }

    // Check if logged in
    const token = sessionStorage.getItem('access_token');
    if (token) {
      fetchUser();
    }
  }, []);

  async function fetchUser() {
    const token = sessionStorage.getItem('access_token');

    const response = await fetch('https://api.example.com/user', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (response.ok) {
      const userData = await response.json();
      setUser(userData);
    }
  }

  return (
    <div>
      {user ? (
        <div>
          <h1>Welcome, {user.name}</h1>
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <button onClick={() => oauth.authorize()}>Login</button>
      )}
    </div>
  );
}
```

---

**Versión:** 1.0.0
