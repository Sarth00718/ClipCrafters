# ClipCrafters 🎬

> AI-Powered Agentic Video Editing Platform — Generate, Edit & Publish stunning videos from plain text in minutes.

[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=white)](https://react.dev)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?logo=mongodb&logoColor=white)](https://mongodb.com)
[![FastAPI](https://img.shields.io/badge/FastAPI-Python-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [System Design](#system-design)
- [ER Diagram](#er-diagram)

---

## Overview

ClipCrafters is a full-stack AI video generation platform where users describe a topic and the system:

1. **Generates a script** using an LLM (via FastAPI AI service)
2. **Splits into scenes** with visual prompts and voiceover text
3. **Renders each scene** into video clips with AI-generated visuals + TTS voice
4. **Stitches clips** into a final polished video
5. **Delivers** the video ready for download or streaming

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENT (React 18)                    │
│         Vite · TailwindCSS · Framer Motion              │
│              Port: 5173  (dev proxy → 5001)             │
└────────────────────────┬────────────────────────────────┘
                         │ REST API (/api)
┌────────────────────────▼────────────────────────────────┐
│               NODE.JS BACKEND (Express)                  │
│   Auth · Projects · Videos · Scenes · Edits             │
│              Port: 5001                                   │
└────────┬───────────────────────────┬────────────────────┘
         │ Mongoose ODM              │ HTTP (Axios)
┌────────▼────────┐        ┌─────────▼──────────────────┐
│   MongoDB Atlas  │        │  FastAPI AI SERVICE (Python)│
│   (Cloud DB)    │        │  Script · Voice · Video gen │
└─────────────────┘        │       Port: 8000            │
                            └────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 18, Vite, TailwindCSS v4, Framer Motion, Axios, React Router DOM, Lucide React, Sonner |
| **Backend** | Node.js 18+, Express.js, MongoDB + Mongoose, JWT, Bcrypt, Cloudinary, Resend |
| **AI Service** | Python, FastAPI, LangChain / OpenAI, ElevenLabs TTS |
| **DevOps** | Nodemon, dotenv, Helmet, Morgan, Rate limiting |

---

## Project Structure

```
ClipCrafters/
├── client/                  # React frontend
│   ├── src/
│   │   ├── components/      # UI, layout, dashboard, editor components
│   │   ├── pages/           # Route-level page components
│   │   ├── context/         # AuthContext, ThemeContext
│   │   ├── hooks/           # Custom React hooks
│   │   ├── services/        # Axios API service modules
│   │   ├── utils/           # Animations, constants, image loader
│   │   └── styles/          # globals.css (design system)
│   ├── index.html
│   ├── vite.config.js       # Vite + API proxy config
│   └── tailwind.config.js
│
├── server/                  # Node.js backend
│   ├── src/
│   │   ├── config/          # env.js, db.js
│   │   ├── controllers/     # Route handler logic
│   │   ├── middlewares/     # Auth, rate limit, upload, error
│   │   ├── models/          # Mongoose schemas
│   │   ├── routes/          # Express routers
│   │   ├── services/        # Business logic (AI calls, email, etc.)
│   │   ├── utils/           # Logger, validators utils
│   │   └── validators/      # Joi/express-validator schemas
│   ├── server.js            # Entry point
│   └── .env                 # Environment variables (not committed)
│
└── README.md                # This file
```

---

## Quick Start

### Prerequisites
- Node.js ≥ 18
- MongoDB Atlas account (or local MongoDB)
- Python 3.10+ (for AI service)

### 1. Clone & Install

```bash
git clone https://github.com/Fenil412/ClipCrafters.git
cd ClipCrafters

# Backend
cd server && npm install

# Frontend
cd ../client && npm install
```

### 2. Configure Environment

```bash
# server/.env — see server/README.md for all variables
cp server/.env.example server/.env

# client — no .env needed (uses Vite proxy)
```

### 3. Run

```bash
# Terminal 1 — Backend
cd server && npm run dev

# Terminal 2 — Frontend
cd client && npm run dev
```

Frontend: http://localhost:5173  
Backend API: http://localhost:5001/api  
Health check: http://localhost:5001/api/health

---

## Environment Variables

See [`server/README.md`](server/README.md#environment-variables) for the full list.

---

## API Reference

Base URL: `http://localhost:5001/api`

All protected routes require: `Authorization: Bearer <token>`

### Authentication

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/auth/register` | ❌ | Register new user |
| `POST` | `/auth/login` | ❌ | Login, returns JWT |
| `GET` | `/auth/me` | ✅ | Get current user |
| `POST` | `/auth/send-otp` | ✅ | Send OTP to email |
| `POST` | `/auth/verify-otp` | ✅ | Verify OTP |

### Projects

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/projects/create` | ✅ | Create new project |
| `GET` | `/projects` | ✅ | Get all user projects |
| `GET` | `/projects/:id` | ✅ | Get single project |
| `PUT` | `/projects/:id` | ✅ | Update project |
| `DELETE` | `/projects/:id` | ✅ | Delete project |

### Videos

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/videos/generate` | ✅ | Generate AI video from project |
| `POST` | `/videos/upload` | ✅ | Upload source video file |
| `GET` | `/videos/:id` | ✅ | Get video details & status |

### Scenes

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/scenes/video/:videoId` | ✅ | Get all scenes for a video |
| `PUT` | `/scenes/:sceneId` | ✅ | Update scene (script/prompt) |

### Edit History

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/edits/create` | ✅ | Record an edit action |
| `GET` | `/edits/scene/:sceneId` | ✅ | Get edit history for a scene |

### System

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/health` | ❌ | Server health check |

---

## System Design

```
User Request → Rate Limiter → Auth Middleware → Controller
                                                    │
                                             Service Layer
                                           ┌──────┴──────┐
                                      MongoDB         FastAPI AI
                                      (persist)       (generate)
                                           └──────┬──────┘
                                            Response ← Controller
```

**Key design decisions:**
- JWT access tokens (1d) + refresh tokens (7d)
- Rate limiting on auth routes (prevent brute force)
- Cloudinary for video/image storage
- Resend for transactional emails
- All AI generation async via FastAPI

---

## ER Diagram

```
User ──────────── has many ──────────── Project
  │                                        │
  │                               has many │
  │                                        ▼
  │                                      Video
  │                                        │
  │                               has many │
  │                                        ▼
  │                                      Scene ── has many ── EditHistory
  └──────── referenced by ───────────────────┘
```

### Collections

| Collection | Key Fields |
|---|---|
| `users` | `name`, `email`, `password (hashed)`, `isVerified`, `refreshToken` |
| `projects` | `userId`, `title`, `topic`, `status`, `videoId` |
| `videos` | `projectId`, `userId`, `status`, `cloudinaryUrl`, `duration` |
| `scenes` | `videoId`, `order`, `script`, `visualPrompt`, `audioUrl`, `clipUrl` |
| `edithistories` | `sceneId`, `userId`, `changeType`, `before`, `after` |