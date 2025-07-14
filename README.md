# 🪞 DevMirror – AI-Powered Developer Journal

**DevMirror** is a full-stack Java application that helps developers document their learning journey, manage code snippets, get AI-based code reviews, and track daily productivity—all secured with JWT authentication.

---

## 🚀 Features

### 📝 Journal
- Write and store daily developer logs
- Track mood and progress over time

### 🧠 AI Code Review
- Paste any code and get improvement suggestions using OpenAI (GPT-3.5)

### 🗂️ Code Snippet Manager
- Save reusable code snippets
- Tag by language, project, or concept
- AI-based snippet review

### 🔐 Secure API with JWT
- User registration and login
- All journal and snippet routes protected via JWT

---

## 🧰 Tech Stack

| Layer       | Tech                          |
|-------------|-------------------------------|
| Backend     | Java 17, Spring Boot, JPA     |
| Auth        | Spring Security, JWT          |
| AI Review   | OpenAI GPT-3.5 API            |
| Database    | PostgreSQL / H2 (local dev)   |
| Deployment  | Docker + Render (optional)    |

---

## ⚙️ Setup Instructions

### 🔧 1. Clone & Build
```bash
git clone https://github.com/jivanshirke1/devmirror.git
cd devmirror
./mvnw clean package
```

### 🔐 2. Configure OpenAI API
In `application.properties`, add:
```
openai.api.key=sk-your-api-key
```

### 🐳 3. (Optional) Docker Build
```bash
docker build -t devmirror-app .
docker run -p 8080:8080 devmirror-app
```

---

## 📡 API Endpoints

### 🔐 Auth
- `POST /api/auth/register`
- `POST /api/auth/login`

### 📓 Journal
- `GET /api/journals`
- `POST /api/journals`
- `DELETE /api/journals/{id}`
- `POST /api/journals/ai/suggest`

### 📘 Snippets
- `GET /api/snippets`
- `POST /api/snippets`
- `DELETE /api/snippets/{id}`
- `POST /api/snippets/ai-review`

> All endpoints require `Authorization: Bearer <token>` header (except `/api/auth/**`)

---

## 🌐 Live Demo

> 🔗 [https://devmirror.onrender.com](https://devmirror.onrender.com)  
Explore the deployed version of DevMirror hosted on Render.

---

## 🧑‍💻 Author

**👨‍💻 Jivan Shirke**  
📧 [jivanshirke2001@gmail.com]
