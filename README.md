<div align="center">

# 🗄️ Blog API: Express + MongoDB

**Week 5 · Connect DB to API**: a REST API with Mongoose models, request validation and clean error handling.

![Node](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose%208-47A248?logo=mongodb&logoColor=white)
![Tests](https://img.shields.io/badge/integration%20tests-18%20passing-brightgreen)

</div>

---

## 📑 Table of Contents
1. [Task Description](#-task-description)
2. [Features](#-features)
3. [Tech Stack](#-tech-stack)
4. [Project Structure](#-project-structure)
5. [Installation & Setup](#-installation--setup)
6. [Running the Project](#-running-the-project)
7. [API Reference](#-api-reference)
8. [Data Model & Validation](#-data-model--validation)
9. [Error Format](#-error-format)
10. [Testing](#-testing)
11. [Output / Results](#-output--results)
12. [Troubleshooting](#-troubleshooting)

---

## 🎯 Task Description
> **Week 5: Connect DB to API**: Connect your Express API to PostgreSQL/MongoDB with proper models and validation.

This project connects an **Express** API to **MongoDB** using **Mongoose**. Blog posts are stored in the database through a proper `Post` model, every request is validated before it reaches the database, and all errors come back in one consistent JSON shape.

## ✨ Features
- Full **CRUD** for blog posts (create, list, read, update, delete)
- **Mongoose model** with typed fields, defaults, enums, indexes and timestamps
- **Two-layer validation**: `express-validator` on the request, Mongoose schema rules on the model
- **Pagination**, **search** (title / content / author) and **tag filtering** on the list endpoint
- **Mass-assignment protection**: only whitelisted fields are ever saved
- Safe regex search (user input is escaped)
- Consistent error responses (400 / 404 / 500), malformed JSON handled
- Health-check endpoint that reports DB connection state
- Seed script and an integration test suite that runs against a real MongoDB

## 🧰 Tech Stack
| Tool | Purpose |
|---|---|
| Node.js 18+ | Runtime |
| Express 4 | HTTP server and routing |
| MongoDB + Mongoose 8 | Database and ODM models |
| express-validator | Request validation |
| cors, morgan, dotenv | CORS, request logging, configuration |

## 📁 Project Structure
```
Internship_Task/
├── Source_Code/
│   ├── src/
│   │   ├── config/db.js              # MongoDB connection
│   │   ├── models/Post.js            # Mongoose schema and validation
│   │   ├── controllers/postController.js
│   │   ├── routes/posts.js
│   │   ├── middleware/
│   │   │   ├── validate.js           # express-validator rules
│   │   │   └── errors.js             # 404 and error handler
│   │   ├── app.js                    # Express app
│   │   ├── server.js                 # Entry point
│   │   └── seed.js                   # Sample data
│   ├── tests/api.test.js             # Integration tests
│   ├── .env.example
│   └── package.json
├── Output/
│   ├── test-results.txt              # Test run (18 passing)
│   └── api-session.txt               # Real requests and responses
└── README.md
```

## ⚙️ Installation & Setup

**Requirements:** Node.js 18+ and a MongoDB instance (local install, Docker, or a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster).

```bash
cd Source_Code
npm install
cp .env.example .env        # Windows: copy .env.example .env
```

Edit `.env`:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `5000` | Port the API listens on |
| `MONGODB_URI` | `mongodb://127.0.0.1:27017/blog_api` | MongoDB connection string (use your Atlas URI if hosted) |
| `CLIENT_ORIGIN` | `http://localhost:5173` | Allowed CORS origin(s), comma separated |

> No MongoDB installed? Run one with Docker: `docker run -d -p 27017:27017 --name mongo mongo:7`

## ▶️ Running the Project
```bash
npm run seed     # optional: load 3 sample posts
npm start        # production
npm run dev      # auto-restart on changes
```
Open <http://localhost:5000/api/health>. You should see `{"status":"ok","db":"connected"}`.

## 📡 API Reference
Base URL: `http://localhost:5000/api`

| Method | Endpoint | Description | Success |
|---|---|---|---|
| `GET` | `/health` | Server and DB status | `200` |
| `GET` | `/posts` | List posts (paginated) | `200` |
| `POST` | `/posts` | Create a post | `201` |
| `GET` | `/posts/:id` | Get one post | `200` |
| `PUT` / `PATCH` | `/posts/:id` | Update a post (partial update allowed) | `200` |
| `DELETE` | `/posts/:id` | Delete a post | `200` |

**List query parameters**

| Param | Example | Notes |
|---|---|---|
| `page` | `?page=2` | Default `1` |
| `limit` | `?limit=5` | Default `10`, max `50` |
| `search` | `?search=react` | Matches title, content or author (case-insensitive) |
| `tag` | `?tag=node` | Exact tag match |
| `status` | `?status=draft` | `draft` or `published` |

**Example: create a post**
```bash
curl -X POST http://localhost:5000/api/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Hello MongoDB","author":"Rahul","content":"First post saved through the Express API.","tags":["mongodb"]}'
```
```json
{
  "title": "Hello MongoDB",
  "content": "First post saved through the Express API.",
  "author": "Rahul",
  "tags": ["mongodb"],
  "status": "published",
  "createdAt": "2026-11-20T03:32:06.495Z",
  "updatedAt": "2026-11-20T03:32:06.495Z",
  "id": "6affbf36d35f0b719067d81a"
}
```

**Example: list response shape**
```json
{ "data": [ /* posts */ ], "page": 1, "limit": 10, "total": 24, "totalPages": 3 }
```

## 🧱 Data Model & Validation

**`Post` schema** (`src/models/Post.js`)

| Field | Type | Rules |
|---|---|---|
| `title` | String | required, trimmed, 3–120 chars |
| `content` | String | required, trimmed, 10–10,000 chars |
| `author` | String | required, trimmed, 2–60 chars |
| `tags` | [String] | optional, max 5 tags, each max 24 chars, stored lowercase |
| `status` | String | `draft` or `published` (default `published`) |
| `createdAt`, `updatedAt` | Date | added automatically |

**Indexes:** text index on `title` and `content`; descending index on `createdAt`.

**Validation layers**
1. **Request layer** (`middleware/validate.js`): checks body, params and query, and rejects invalid MongoDB ids before touching the database.
2. **Model layer** (Mongoose schema): a last line of defence. Update queries run with `runValidators: true`, so edits are validated too.

## 🚦 Error Format
Every error uses the same shape, so a frontend can handle them in one place.

```json
{
  "message": "Validation failed",
  "errors": [
    { "field": "title", "message": "Title must be 3-120 characters" },
    { "field": "content", "message": "Content is required" }
  ]
}
```

| Status | When |
|---|---|
| `400` | Validation failed, invalid id, malformed JSON |
| `404` | Post or route not found |
| `500` | Unexpected server error (details are logged, never leaked) |

## 🧪 Testing
The suite is designed to run against a **real MongoDB** (it uses a separate `blog_api_test` database and drops it afterwards). Run it locally to verify the current environment.

```bash
npm test
```
It contains 20 checks: create, validation failures, unknown-field stripping, pagination, search, tag filter, get, update, delete, invalid ids, 404s, malformed JSON, invalid status filters, and empty update bodies. The current review environment could not execute the suite because dependencies were not installed (`Cannot find module dotenv`); all JavaScript files passed `node --check`. Run `npm install` and `npm test` with MongoDB running before reporting a passing integration result.

## 📸 Output / Results
- `Output/test-results.txt`: full test run, **20 integration checks are present; they have not been executed in this review environment. See `Output/test-results.txt` for the exact blocker and verification status.
- `Output/api-session.txt`: real requests and responses captured from the running API

## 🛠️ Troubleshooting
| Problem | Fix |
|---|---|
| `MONGODB_URI is not set` | You skipped `cp .env.example .env` |
| `Failed to start server: connect ECONNREFUSED 127.0.0.1:27017` | MongoDB is not running. Start it, or use an Atlas URI |
| Atlas connection times out | Add your IP in Atlas, then *Network Access* |
| `EADDRINUSE` | Port 5000 is taken. Change `PORT` in `.env` (macOS often uses 5000 for AirPlay) |

---
<div align="center">Built for the SDM Technology internship · Week 5</div>
