
# 📝 Simple Note MERN App – Full Documentation

This guide walks you through building a complete CRUD (Create, Read, Update, Delete) Note App using the **MERN Stack**.

---

## ✅ Requirements

- Node.js (v18+ recommended)
- MongoDB (Local or Atlas)
- Code Editor (VS Code)
- Terminal / Command Prompt

---

## 📁 Project Structure Overview

```
simple-note-mern-app/
├── backend/
│   ├── models/Note.js
│   ├── routes/noteRoutes.js
│   ├── server.js
│   └── .env
│
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── index.js
    │   └── components/
    │       └── NoteApp.jsx
    └── vite.config.js
```

---

## 🔧 Step-by-Step Project Setup

### 🧩 Step 1: Initialize Backend

```bash
mkdir simple-note-mern-app
cd simple-note-mern-app
mkdir backend && cd backend
npm init -y
npm install express mongoose cors dotenv
```

### 🖥️ Create backend/server.js

```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

// Routes
const noteRoutes = require('./routes/noteRoutes');
app.use('/api/notes', noteRoutes);

// Connect DB & Start Server
mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    app.listen(process.env.PORT || 5000, () =>
      console.log('Server started on port 5000')
    );
  })
  .catch((err) => console.error(err));
```

### 📄 Create .env file

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/noteapp
```

---

### 📚 Step 2: Create Note Model – `backend/models/Note.js`

```js
const mongoose = require('mongoose');

const noteSchema = new mongoose.Schema(
  {
    title: String,
    description: String,
  },
  { timestamps: true }
);

module.exports = mongoose.model('Note', noteSchema);
```

---

### 🌐 Step 3: Create Note Routes – `backend/routes/noteRoutes.js`

```js
const express = require('express');
const router = express.Router();
const Note = require('../models/Note');

// Get all notes
router.get('/', async (req, res) => {
  const notes = await Note.find().sort({ createdAt: -1 });
  res.json(notes);
});

// Create a note
router.post('/', async (req, res) => {
  const { title, description } = req.body;
  const note = await Note.create({ title, description });
  res.json(note);
});

// Update a note
router.put('/:id', async (req, res) => {
  const note = await Note.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(note);
});

// Delete a note
router.delete('/:id', async (req, res) => {
  await Note.findByIdAndDelete(req.params.id);
  res.json({ success: true });
});

module.exports = router;
```

---

## ⚛️ Step 4: Set Up Frontend (React + Vite)

```bash
cd ..
npm create vite@latest frontend --template react
cd frontend
npm install axios
```

### 📂 Folder Structure

```
frontend/
├── src/
│   ├── App.jsx
│   ├── index.js
│   └── components/
│       └── NoteApp.jsx
```

---

### 🔧 Edit `src/App.jsx`

```jsx
import NoteApp from "./components/NoteApp";
export default function App() {
  return <NoteApp />;
}
```

---

### 📄 Create `NoteApp.jsx`

```jsx
import { useEffect, useState } from "react";
import axios from "axios";

export default function NoteApp() {
  const [formData, setFormData] = useState({ title: "", description: "" });
  const [notes, setNotes] = useState([]);
  const [editId, setEditId] = useState(null);

  const fetchNotes = async () => {
    const res = await axios.get("http://localhost:5000/api/notes");
    setNotes(res.data);
  };

  useEffect(() => {
    fetchNotes();
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (editId) {
      await axios.put(`http://localhost:5000/api/notes/${editId}`, formData);
      setEditId(null);
    } else {
      await axios.post("http://localhost:5000/api/notes", formData);
    }
    setFormData({ title: "", description: "" });
    fetchNotes();
  };

  const handleDelete = async (id) => {
    await axios.delete(`http://localhost:5000/api/notes/${id}`);
    fetchNotes();
  };

  const handleEdit = (note) => {
    setFormData({ title: note.title, description: note.description });
    setEditId(note._id);
  };

  return (
    <div style={{ padding: "2rem", maxWidth: "600px", margin: "auto" }}>
      <h2>📝 Note Manager</h2>
      <form onSubmit={handleSubmit}>
        <input
          placeholder="Title"
          value={formData.title}
          onChange={(e) => setFormData({ ...formData, title: e.target.value })}
          required
        />
        <textarea
          placeholder="Description"
          value={formData.description}
          onChange={(e) =>
            setFormData({ ...formData, description: e.target.value })
          }
          required
        />
        <button type="submit">{editId ? "Update" : "Add"} Note</button>
      </form>

      <hr />
      <ul>
        {notes.map((note) => (
          <li key={note._id} style={{ marginBottom: "1rem" }}>
            <strong>{note.title}</strong>: {note.description}
            <br />
            <button onClick={() => handleEdit(note)}>Edit</button>
            <button onClick={() => handleDelete(note._id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## ✅ Step 5: Run the Project

### Start Backend

```bash
cd backend
node server.js
```

### Start Frontend

```bash
cd frontend
npm run dev
```

Visit 👉 http://localhost:5173

---

## 🌐 Optional: Deploy Project

### 🌍 Backend on Render

1. Go to https://render.com
2. Create a new web service and connect your GitHub repo
3. Add `MONGO_URI` in environment variables
4. Deploy

### 🌐 Frontend on Vercel

1. Go to https://vercel.com
2. Import your frontend repo
3. Set build command: `npm run build`
4. Set output folder: `dist`
5. Deploy

---

## 🧹 Final Tips

- 🧪 Test each API using Postman before connecting to frontend.
- 📦 Use MongoDB Atlas for cloud database.
- 🎨 Add styling (Tailwind CSS or CSS modules) for a better UI.
