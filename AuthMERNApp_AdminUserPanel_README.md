
# MERN Auth App with Admin & User Panels – Full Documentation

A secure authentication system with role-based access control using the MERN stack (MongoDB, Express.js, React.js, Node.js). This project features both an **Admin Panel** and a **User Panel**.

---

## ✅ Features

- User registration and login
- Password hashing with bcryptjs
- JWT-based authentication
- Role-based access (`user`, `admin`)
- Protected routes
- Separate dashboards for admin and user

---

## 🧱 Models Update

### `models/User.js`
```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['user', 'admin'], default: 'user' }
});

module.exports = mongoose.model('User', userSchema);
```

---

## 🔐 Middleware for Auth & Role

### `middleware/auth.js`
```js
const jwt = require('jsonwebtoken');

exports.authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Unauthorized' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ message: 'Invalid token' });
  }
};

exports.adminMiddleware = (req, res, next) => {
  if (req.user.role !== 'admin') return res.status(403).json({ message: 'Access denied' });
  next();
};
```

---

## 🧠 Backend Route Example

### `routes/userRoutes.js`
```js
const express = require('express');
const { authMiddleware, adminMiddleware } = require('../middleware/auth');
const router = express.Router();

router.get('/user-data', authMiddleware, (req, res) => {
  res.json({ message: 'Hello User', user: req.user });
});

router.get('/admin-data', authMiddleware, adminMiddleware, (req, res) => {
  res.json({ message: 'Hello Admin', user: req.user });
});

module.exports = router;
```

### Attach route in `server.js`
```js
const userRoutes = require('./routes/userRoutes');
app.use('/api/user', userRoutes);
```

---

## 🌐 Frontend Panel Logic

### `App.jsx`
```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function App() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const token = localStorage.getItem('token');
    axios.get('http://localhost:5000/api/user/user-data', {
      headers: { Authorization: `Bearer ${token}` }
    }).then(res => {
      setUser(res.data.user);
    }).catch(() => {
      setUser(null);
    });
  }, []);

  if (!user) return <h2>Loading...</h2>;
  if (user.role === 'admin') return <AdminPanel />;
  return <UserPanel />;
}

function AdminPanel() {
  return <h2>Welcome Admin</h2>;
}

function UserPanel() {
  return <h2>Welcome User</h2>;
}
```

---

## ✅ Run the App

- **Register** a user via `/register` route
- Use MongoDB Compass or code to manually update role to `admin` for admin access
- **Login**, store token in `localStorage`
- Open app: Admins see admin panel, users see user panel

---

## 🔐 Future Improvements

- Admin can promote/demote users
- Add logout, refresh token flow
- Use `React Context` or `Redux` for auth state
- Protect frontend routes with `react-router-dom` and role logic

---
