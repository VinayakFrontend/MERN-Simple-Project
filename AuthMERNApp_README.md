
# Simple Sign In / Sign Up MERN Project – Full Documentation

This is a basic authentication system using the MERN stack: MongoDB, Express.js, React.js, and Node.js. It supports user registration and login with password hashing and JWT authentication.

---

## ✅ Requirements

- Node.js (v18+)
- MongoDB (local or MongoDB Atlas)
- VS Code
- Postman (for API testing)
- Browser (for frontend)

---

## 📁 Project Structure

```
auth-mern-app/
├── backend/
│   ├── models/User.js
│   ├── routes/authRoutes.js
│   ├── controllers/authController.js
│   ├── server.js
│   └── .env
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── index.js
    │   └── pages/
    │       ├── Login.jsx
    │       └── Register.jsx
    └── vite.config.js
```

---

## 🛠️ Backend Setup

### Step 1: Initialize backend

```bash
mkdir auth-mern-app
cd auth-mern-app
mkdir backend && cd backend
npm init -y
npm install express mongoose cors dotenv bcryptjs jsonwebtoken
```

### Step 2: Backend files

**server.js**
```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const authRoutes = require('./routes/authRoutes');
app.use('/api/auth', authRoutes);

mongoose.connect(process.env.MONGO_URI)
  .then(() => app.listen(process.env.PORT, () => console.log('Server running')))
  .catch(err => console.log(err));
```

**.env**
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/authdb
JWT_SECRET=your_jwt_secret
```

**models/User.js**
```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true }
});

module.exports = mongoose.model('User', userSchema);
```

**controllers/authController.js**
```js
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

exports.register = async (req, res) => {
  const { name, email, password } = req.body;
  const hash = await bcrypt.hash(password, 10);
  const user = await User.create({ name, email, password: hash });
  res.json(user);
};

exports.login = async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user || !(await bcrypt.compare(password, user.password)))
    return res.status(401).json({ error: 'Invalid credentials' });

  const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET);
  res.json({ token });
};
```

**routes/authRoutes.js**
```js
const express = require('express');
const { register, login } = require('../controllers/authController');
const router = express.Router();

router.post('/register', register);
router.post('/login', login);

module.exports = router;
```

---

## 🛠️ Frontend Setup

### Step 3: Initialize frontend

```bash
cd ..
npm create vite@latest frontend --template react
cd frontend
npm install axios react-router-dom
```

**src/main.jsx**
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import App from './App';
import Login from './pages/Login';
import Register from './pages/Register';

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<App />} />
      <Route path="/login" element={<Login />} />
      <Route path="/register" element={<Register />} />
    </Routes>
  </BrowserRouter>
);
```

**src/App.jsx**
```jsx
export default function App() {
  return <h2>Welcome to Auth App</h2>;
}
```

**src/pages/Register.jsx**
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

**src/pages/Login.jsx**
```jsx
import { useState } from "react";
import axios from "axios";

export default function Login() {
  const [form, setForm] = useState({ email: "", password: "" });

  const handleSubmit = async (e) => {
    e.preventDefault();
    const res = await axios.post("http://localhost:5000/api/auth/login", form);
    alert("Token: " + res.data.token);
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

## ✅ Run the App

**Start backend**
```bash
cd backend
node server.js
```

**Start frontend**
```bash
cd frontend
npm run dev
```

Visit: http://localhost:5173

---

## 📝 Notes

- Use Postman to test `/api/auth/register` and `/api/auth/login`
- JWT tokens can be stored in `localStorage` or `cookies`
- Use `Authorization` headers for protected routes

---

