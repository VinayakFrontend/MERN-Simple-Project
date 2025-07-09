
# 📁 Simple MERN File Upload & Download App – Full Guide

This is a step-by-step guide to build a simple **MERN** (MongoDB, Express, React, Node.js) project that allows users to upload and download files.

---

## 📦 Tech Stack

- **MongoDB** – Stores file metadata
- **Express.js** – Handles backend APIs
- **Multer** – Middleware for handling multipart/form-data
- **React** – Frontend file upload & download UI
- **Axios** – Makes HTTP requests from React

---

## 🗂️ 1. Project Structure

```
mern-file-app/
├── backend/
│   ├── models/File.js
│   ├── routes/fileRoutes.js
│   ├── server.js
├── frontend/
│   └── src/
│       ├── App.jsx
│       └── index.js
```

---

## 🛠️ 2. Backend Setup

### 📌 Install Dependencies
```bash
cd backend
npm init -y
npm install express mongoose multer cors
```

### 📁 `models/File.js`
```js
const mongoose = require('mongoose');

const fileSchema = new mongoose.Schema({
  filename: String,
  originalname: String,
  path: String,
  mimetype: String,
  size: Number,
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('File', fileSchema);
```

### 🔁 `routes/fileRoutes.js`
```js
const express = require('express');
const multer = require('multer');
const File = require('../models/File');
const router = express.Router();
const path = require('path');

const upload = multer({ dest: 'uploads/' });

router.post('/upload', upload.single('file'), async (req, res) => {
  const file = await File.create(req.file);
  res.json({ id: file._id });
});

router.get('/download/:id', async (req, res) => {
  const file = await File.findById(req.params.id);
  res.download(path.resolve(file.path), file.originalname);
});

router.get('/files', async (req, res) => {
  const files = await File.find().sort({ createdAt: -1 });
  res.json(files);
});

module.exports = router;
```

### 🚀 `server.js`
```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const fileRoutes = require('./routes/fileRoutes');
const app = express();

app.use(cors());
app.use(express.json());
app.use('/uploads', express.static('uploads'));
app.use('/api', fileRoutes);

mongoose.connect('mongodb://localhost:27017/filesdb')
  .then(() => app.listen(5000, () => console.log('Server running on port 5000')))
  .catch(err => console.error(err));
```

---

## 🌐 3. Frontend Setup

### 📌 Setup
```bash
cd ../frontend
npm create vite@latest
cd frontend && npm install axios
```

### 🔁 `src/App.jsx`
```jsx
import { useEffect, useState } from "react";
import axios from "axios";

export default function App() {
  const [file, setFile] = useState(null);
  const [files, setFiles] = useState([]);

  const fetchFiles = async () => {
    const res = await axios.get('http://localhost:5000/api/files');
    setFiles(res.data);
  };

  useEffect(() => { fetchFiles(); }, []);

  const upload = async () => {
    const formData = new FormData();
    formData.append("file", file);
    await axios.post('http://localhost:5000/api/upload', formData);
    fetchFiles();
  };

  return (
    <div>
      <h2>Upload File</h2>
      <input type="file" onChange={e => setFile(e.target.files[0])} />
      <button onClick={upload}>Upload</button>

      <h3>Uploaded Files</h3>
      <ul>
        {files.map(f => (
          <li key={f._id}>
            {f.originalname} - <a href={`http://localhost:5000/api/download/${f._id}`}>Download</a>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## ✅ 4. Run the App

### 🔹 Start Backend
```bash
cd backend
node server.js
```

### 🔹 Start Frontend
```bash
cd frontend
npm run dev
```

---

## 🧾 Features

- Upload any type of file
- Store file metadata in MongoDB
- Download files by clicking link
- File storage in `/uploads`

---

## 🛠️ Optional Improvements

- Store files in cloud (S3, Cloudinary)
- Add file previews (PDF, images)
- Add user authentication
- Delete files from frontend

---

© 2025 MERN File Upload Project by ChatGPT
