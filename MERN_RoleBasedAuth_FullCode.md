
# MERN Auth App with Admin, Employee & User Panels – Full Code & Documentation

This is a complete role-based authentication system built with the MERN stack. It includes:

- ✅ User registration & login
- 🔐 JWT authentication
- 🧑‍💻 Role-based dashboards for **Admin**, **Employee**, and **User**

---

## 📁 Project Structure

```
auth-mern-app/
├── backend/
│   ├── models/User.js
│   ├── routes/authRoutes.js
│   ├── routes/dashboardRoutes.js
│   ├── middleware/auth.js
│   ├── controllers/authController.js
│   ├── server.js
│   └── .env
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── index.js
    │   └── pages/
    │       ├── Login.jsx
    │       ├── Register.jsx
    └── vite.config.js
```

---

## 🔧 Backend Setup

### `.env`
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/authrolesdb
JWT_SECRET=your_secret_key
```

### `server.js`

```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const authRoutes = require('./routes/authRoutes');
const dashboardRoutes = require('./routes/dashboardRoutes');

app.use('/api/auth', authRoutes);
app.use('/api/dashboard', dashboardRoutes);

mongoose.connect(process.env.MONGO_URI)
  .then(() => app.listen(process.env.PORT, () => console.log('Server running')))
  .catch(err => console.error(err));
```

### `models/User.js`

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  role: { type: String, enum: ['user', 'employee', 'admin'], default: 'user' }
});

module.exports = mongoose.model('User', userSchema);
```

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

exports.roleMiddleware = (roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ message: 'Access denied' });
  }
  next();
};
```

### `controllers/authController.js`

```js
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

exports.register = async (req, res) => {
  const { name, email, password } = req.body;
  const existing = await User.findOne({ email });
  if (existing) return res.status(400).json({ error: 'Email already exists' });

  const hash = await bcrypt.hash(password, 10);
  const user = await User.create({ name, email, password: hash });
  res.json(user);
};

exports.login = async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user || !(await bcrypt.compare(password, user.password)))
    return res.status(401).json({ error: 'Invalid credentials' });

  const token = jwt.sign({ userId: user._id, role: user.role }, process.env.JWT_SECRET);
  res.json({ token });
};
```

### `routes/authRoutes.js`

```js
const express = require('express');
const { register, login } = require('../controllers/authController');
const router = express.Router();

router.post('/register', register);
router.post('/login', login);

module.exports = router;
```

### `routes/dashboardRoutes.js`

```js
const express = require('express');
const { authMiddleware, roleMiddleware } = require('../middleware/auth');
const router = express.Router();

router.get('/user', authMiddleware, roleMiddleware(['user']), (req, res) => {
  res.json({ message: 'User dashboard' });
});

router.get('/employee', authMiddleware, roleMiddleware(['employee']), (req, res) => {
  res.json({ message: 'Employee dashboard' });
});

router.get('/admin', authMiddleware, roleMiddleware(['admin']), (req, res) => {
  res.json({ message: 'Admin dashboard' });
});

module.exports = router;
```

---

## 🌐 Frontend Setup

### `App.jsx`

```jsx
import { useEffect, useState } from 'react';
import axios from 'axios';

export default function App() {
  const [role, setRole] = useState(null);

  useEffect(() => {
    const token = localStorage.getItem('token');

    axios.get('http://localhost:5000/api/dashboard/admin', {
      headers: { Authorization: `Bearer ${token}` }
    }).then(() => setRole('admin')).catch(() => {});

    axios.get('http://localhost:5000/api/dashboard/employee', {
      headers: { Authorization: `Bearer ${token}` }
    }).then(() => setRole('employee')).catch(() => {});

    axios.get('http://localhost:5000/api/dashboard/user', {
      headers: { Authorization: `Bearer ${token}` }
    }).then(() => setRole('user')).catch(() => {});
  }, []);

  if (!role) return <h2>Loading...</h2>;
  if (role === 'admin') return <h2>Admin Panel</h2>;
  if (role === 'employee') return <h2>Employee Panel</h2>;
  return <h2>User Panel</h2>;
}
```

### `pages/Register.jsx`

```jsx
import { useState } from "react";
import axios from "axios";

export default function Register() {
  const [form, setForm] = useState({ name: "", email: "", password: "" });

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post("http://localhost:5000/api/auth/register", form);
    alert("Registered successfully");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Name" onChange={e => setForm({...form, name: e.target.value})} />
      <input placeholder="Email" onChange={e => setForm({...form, email: e.target.value})} />
      <input placeholder="Password" type="password" onChange={e => setForm({...form, password: e.target.value})} />
      <button type="submit">Register</button>
    </form>
  );
}
```

### `pages/Login.jsx`

```jsx
import { useState } from "react";
import axios from "axios";

export default function Login() {
  const [form, setForm] = useState({ email: "", password: "" });

  const handleSubmit = async (e) => {
    e.preventDefault();
    const res = await axios.post("http://localhost:5000/api/auth/login", form);
    localStorage.setItem("token", res.data.token);
    alert("Logged in!");
    window.location.reload();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Email" onChange={e => setForm({...form, email: e.target.value})} />
      <input placeholder="Password" type="password" onChange={e => setForm({...form, password: e.target.value})} />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## ▶️ Run the App

### Backend

```bash
cd backend
node server.js
```

### Frontend

```bash
cd frontend
npm run dev
```

---

## 🛠️ Optional Enhancements

- Admin can manage all users
- Employees handle data assigned by admins
- React Router + Role Protected Routes
- UI improvements using Tailwind CSS or Bootstrap
