# its-new
just for m
📦 fastapi-react-todo
├─ backen
│  ├─ ma
│  ├─ databas
│  ├─ requirements.t
│  └─ tes
│     └─ test_api.
├─ fronte
│  ├─ package.js
│  ├─ vite.config.
│  └─ s
│     ├─ App.jsx
│     └─ api.j
├─ README.md
├─ LICENS
├─ .gitigno
└─ .github/
   └─ workflows/
      └─ ci.yml

========================
README.md
========================
# FastAPI + React Todo App

A minimal full-stack app:
- **Backend:** FastAPI with SQLite
- **Frontend:** React (Vite + Tailwind)
- **Tests + GitHub Actions*

## Run backend
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

## Run frontend
```bash
cd frontend
npm install
npm run dev
```

## License
MIT

========================
LICENSE
========================
MIT License

Copyright (c) 2025 <Your Name>

Permission is hereby granted, free of charge, to any person obtaining a copy
...

========================
.gitignore
========================
# Python
__pycache__/
.venv/
*.py[cod]

# Node
node_modules/
dist/
.env

# IDEs
.vscode/
.idea/
.DS_Store

========================
backend/requirements.txt
========================
fastapi
uvicorn
sqlalchemy
pytest
httpx

========================
backend/database.py
========================
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

SQLALCHEMY_DATABASE_URL = "sqlite:///./todos.db"

engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

========================
backend/models.py
========================
from sqlalchemy import Column, Integer, String, Boolean
from .database import Base

class Todo(Base):
    __tablename__ = "todos"
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    done = Column(Boolean, default=False)

========================
backend/main.py
========================
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from . import models, database

models.Base.metadata.create_all(bind=database.engine)

app = FastAPI(title="Todo API")

@app.get("/todos")
def list_todos(db: Session = Depends(database.get_db)):
    return db.query(models.Todo).all()

@app.post("/todos")
def create_todo(title: str, db: Session = Depends(database.get_db)):
    todo = models.Todo(title=title, done=False)
    db.add(todo)
    db.commit()
    db.refresh(todo)
    return todo

@app.put("/todos/{todo_id}")
def toggle(todo_id: int, db: Session = Depends(database.get_db)):
    todo = db.query(models.Todo).get(todo_id)
    todo.done = not todo.done
    db.commit()
    return todo

@app.delete("/todos/{todo_id}")
def delete(todo_id: int, db: Session = Depends(database.get_db)):
    todo = db.query(models.Todo).get(todo_id)
    db.delete(todo)
    db.commit()
    return {"ok": True}

========================
backend/tests/test_api.py
========================
from fastapi.testclient import TestClient
from backend.main import app

client = TestClient(app)

def test_create_and_list():
    r = client.post("/todos", params={"title": "Test"})
    assert r.status_code == 200
    todo = r.json()
    assert todo["title"] == "Test"

    r2 = client.get("/todos")
    assert r2.status_code == 200
    assert any(t["title"] == "Test" for t in r2.json())

========================
frontend/package.json
========================
{
  "name": "todo-frontend",
  "version": "0.1.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "axios": "^1.7.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "tailwindcss": "^3.4.0",
    "vite": "^5.2.0"
  }
}

========================
frontend/vite.config.js
========================
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: { port: 5173 }
})

========================
frontend/src/api.js
========================
import axios from "axios";

const api = axios.create({
  baseURL: "http://127.0.0.1:8000",
});

export const getTodos = () => api.get("/todos");
export const addTodo = (title) => api.post("/todos", null, { params: { title } });
export const toggleTodo = (id) => api.put(`/todos/${id}`);
export const deleteTodo = (id) => api.delete(`/todos/${id}`);

========================
frontend/src/App.jsx
========================
import { useEffect, useState } from "react";
import { getTodos, addTodo, toggleTodo, deleteTodo } from "./api";

export default function App() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState("");

  const load = async () => {
    const res = await getTodos();
    setTodos(res.data);
  };

  useEffect(() => { load(); }, []);

  const handleAdd = async () => {
    if (!text) return;
    await addTodo(text);
    setText("");
    load();
  };

  return (
    <div className="p-6 max-w-lg mx-auto">
      <h1 className="text-2xl font-bold mb-4">Todo List</h1>
      <div className="flex gap-2 mb-4">
        <input
          className="border p-2 flex-1"
          value={text}
          onChange={(e) => setText(e.target.value)}
        />
        <button className="bg-blue-500 text-white px-4 py-2 rounded" onClick={handleAdd}>
          Add
        </button>
      </div>
      <ul>
        {todos.map((t) => (
          <li key={t.id} className="flex justify-between items-center border-b py-2">
            <span
              onClick={() => { toggleTodo(t.id).then(load); }}
              className={t.done ? "line-through cursor-pointer" : "cursor-pointer"}
            >
              {t.title}
            </span>
            <button
              className="text-red-500"
              onClick={() => { deleteTodo(t.id).then(load); }}
            >
              ✕
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

========================
.github/workflows/ci.yml
========================
name: CI

on:
  push:
  pull_request:

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: |
          cd backend
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pytest -q
  frontend-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: |
          cd frontend
          npm install
          npm run build
