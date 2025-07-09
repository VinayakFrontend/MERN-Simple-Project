
# To-Do List MERN App – Full Documentation (Start to Finish)

A full-stack To-Do List App built using the MERN stack: MongoDB, Express.js, React.js, and Node.js. Users can add, update, mark complete/incomplete, and delete tasks.

---

## ✅ Requirements

- Node.js (v18+)
- MongoDB (local or MongoDB Atlas)
- Git + GitHub
- VS Code
- Browser (for frontend testing)

---

## 📁 Project Structure

```
todo-mern-app/
├── backend/
│   ├── models/Task.js
│   ├── routes/taskRoutes.js
│   ├── server.js
│   └── .env
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── index.js
    │   └── components/
    │       └── TodoApp.jsx
    └── vite.config.js
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Initialize Backend

```bash
mkdir todo-mern-app
cd todo-mern-app
mkdir backend && cd backend
npm init -y
npm install express mongoose cors dotenv
```

**backend/server.js**
```js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const taskRoutes = require('./routes/taskRoutes');
app.use('/api/tasks', taskRoutes);

mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    app.listen(process.env.PORT || 5000, () =>
      console.log('Server started on port 5000')
    );
  })
  .catch((err) => console.error(err));
```

**.env**
```
PORT=5000
MONGO_URI=mongodb://localhost:27017/tododb
```

**models/Task.js**
```js
const mongoose = require('mongoose');

const taskSchema = new mongoose.Schema({
  title: { type: String, required: true },
  completed: { type: Boolean, default: false },
});

module.exports = mongoose.model('Task', taskSchema);
```

**routes/taskRoutes.js**
```js
const express = require('express');
const router = express.Router();
const Task = require('../models/Task');

router.get('/', async (req, res) => {
  const tasks = await Task.find();
  res.json(tasks);
});

router.post('/', async (req, res) => {
  const task = await Task.create({ title: req.body.title });
  res.json(task);
});

router.put('/:id', async (req, res) => {
  const updated = await Task.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(updated);
});

router.delete('/:id', async (req, res) => {
  await Task.findByIdAndDelete(req.params.id);
  res.json({ success: true });
});

module.exports = router;
```

---

### Step 2: Initialize Frontend

```bash
cd ..
npm create vite@latest frontend --template react
cd frontend
npm install axios
```

**src/App.jsx**
```jsx
import TodoApp from './components/TodoApp';
export default function App() {
  return <TodoApp />;
}
```

**src/components/TodoApp.jsx**
```jsx
import { useEffect, useState } from "react";
import axios from "axios";

export default function TodoApp() {
  const [task, setTask] = useState("");
  const [tasks, setTasks] = useState([]);

  const fetchTasks = async () => {
    const res = await axios.get("http://localhost:5000/api/tasks");
    setTasks(res.data);
  };

  useEffect(() => {
    fetchTasks();
  }, []);

  const handleAdd = async (e) => {
    e.preventDefault();
    if (task.trim()) {
      await axios.post("http://localhost:5000/api/tasks", { title: task });
      setTask("");
      fetchTasks();
    }
  };

  const handleToggle = async (id, completed) => {
    await axios.put(`http://localhost:5000/api/tasks/${id}`, { completed: !completed });
    fetchTasks();
  };

  const handleDelete = async (id) => {
    await axios.delete(`http://localhost:5000/api/tasks/${id}`);
    fetchTasks();
  };

  return (
    <div style={{ maxWidth: "600px", margin: "auto", padding: "2rem" }}>
      <h2>To-Do List</h2>
      <form onSubmit={handleAdd}>
        <input
          value={task}
          onChange={(e) => setTask(e.target.value)}
          placeholder="Enter task..."
        />
        <button type="submit">Add</button>
      </form>
      <ul>
        {tasks.map((t) => (
          <li key={t._id} style={{ margin: "10px 0" }}>
            <input
              type="checkbox"
              checked={t.completed}
              onChange={() => handleToggle(t._id, t.completed)}
            />
            <span style={{ textDecoration: t.completed ? "line-through" : "none" }}>
              {t.title}
            </span>
            <button onClick={() => handleDelete(t._id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### ✅ Step 3: Run the App

**Start Backend**
```bash
cd backend
node server.js
```

**Start Frontend**
```bash
cd frontend
npm run dev
```

Visit: http://localhost:5173

---

## 🌐 Deployment

**Backend on Render**
- Push backend to GitHub
- Go to https://render.com
- Create Web Service
- Add MONGO_URI in Environment Variables
- Start command: `node server.js`

**Frontend on Vercel**
- Push frontend to GitHub
- Go to https://vercel.com
- Import project
- Set output folder to `dist`
- Set build command to `npm run build`

---

## 🔚 Final Notes

- Add loading states and error handling
- Use Tailwind or Bootstrap for styling
- Add login system with JWT for user-specific tasks
