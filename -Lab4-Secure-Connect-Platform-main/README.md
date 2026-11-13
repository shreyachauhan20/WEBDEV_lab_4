  # Secure Connect Platform

[![Node.js](https://img.shields.io/badge/Node.js-v18+-339933?style=flat&logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-5.x-000000?style=flat&logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?style=flat&logo=mongodb&logoColor=white)](https://mongoosejs.com/)
[![Socket.io](https://img.shields.io/badge/Socket.io-4.x-010101?style=flat&logo=socket.io&logoColor=white)](https://socket.io/)
[![JWT](https://img.shields.io/badge/Auth-JWT%20%26%20Bcrypt-black?style=flat&logo=jsonwebtokens&logoColor=white)](https://jwt.io/)

A full-featured, secure backend REST API and real-time communication platform built with Node.js, Express, MongoDB/Mongoose, and Socket.IO. The platform features robust JSON Web Token (JWT) authentication, role-based access control (RBAC), user-isolated CRUD note management, internal event-driven auditing via Node.js EventEmitter, and authenticated room-based real-time chat.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Configuration](#environment-configuration)
  - [Running the Server](#running-the-server)
- [API Documentation](#api-documentation)
  - [Authentication Endpoints](#authentication-endpoints)
  - [Notes Endpoints](#notes-endpoints)
  - [Health Check](#health-check)
- [Real-Time Communication (Socket.IO)](#real-time-communication-socketio)
- [Event-Driven Architecture](#event-driven-architecture)
- [Testing](#testing)

---

## Features

- **Authentication & Security**
  - User registration with password hashing via `bcrypt` (10 salt rounds).
  - Validation checks for required fields, unique email addresses, and minimum password lengths.
  - Token-based stateless authentication using JSON Web Tokens (JWT) with 1-day expiration.
- **Role-Based Access Control (RBAC)**
  - Support for `user` and `admin` roles.
  - Route guards (`authMiddleware` and `adminMiddleware`) protecting sensitive endpoints.
- **Notes Management (CRUD REST API)**
  - Complete Create, Read, Update, and Delete operations.
  - Data isolation: Regular users can only access, modify, or delete their own notes.
  - Admin privileged access: Admins can view all notes across the platform with populated user details (`username`, `email`).
- **Real-Time Room-Based Chat (Socket.IO)**
  - JWT handshake authentication middleware on WebSocket connections.
  - Dynamic room joining (`joinRoom`).
  - Broadcast room messaging (`chatMessage`) with sender metadata and timestamps.
- **Internal Event-Driven Logging**
  - Node.js `EventEmitter` integration logging note lifecycle operations (`noteCreated`, `noteUpdated`, `noteDeleted`).

---

## Tech Stack

- **Runtime:** [Node.js](https://nodejs.org/)
- **Framework:** [Express 5](https://expressjs.com/)
- **Database:** [MongoDB](https://www.mongodb.com/) with [Mongoose ODM](https://mongoosejs.com/)
- **Real-Time Engine:** [Socket.IO](https://socket.io/)
- **Security:** [Bcrypt](https://github.com/kelektiv/node.bcrypt.js) & [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)
- **Environment Management:** [dotenv](https://github.com/motdotla/dotenv)
- **Development Tooling:** [Nodemon](https://nodemon.io/)

---

## Project Structure

```text
.
├── README.md
└── WEBDEV_lab4/
    ├── .env.example                     # Sample environment variables
    ├── .gitignore                       # Git ignore file (node_modules, .env)
    ├── package.json                     # Project dependencies and npm scripts
    ├── package-lock.json
    ├── server.js                        # App entry point, Socket.IO & DB setup
    ├── middleware/
    │   └── authMiddleware.js            # JWT auth and Admin RBAC middlewares
    ├── models/
    │   ├── Note.js                      # Note schema (title, content, owner ref)
    │   └── User.js                      # User schema (username, email, password, role)
    ├── routes/
    │   ├── authRoutes.js                # /api/auth routes (signup, login)
    │   └── noteRoutes.js                # /api/notes routes (CRUD + admin views)
    └── test on thunderclient/           # API testing screenshots
```

---

## Getting Started

### Prerequisites

Ensure you have the following installed on your system:
- [Node.js](https://nodejs.org/) (v18.x or later recommended)
- [npm](https://www.npmjs.com/) (bundled with Node.js)
- [MongoDB](https://www.mongodb.com/try/download/community) installed locally or a [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) cloud URI

### Installation

1. Navigate to the project directory:
   ```bash
   cd WEBDEV_lab4
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

### Environment Configuration

Create a `.env` file in the `WEBDEV_lab4` directory based on `.env.example`:

```bash
cp .env.example .env
```

Populate the following environment variables:

| Variable | Description | Example |
| :--- | :--- | :--- |
| `PORT` | Port number the HTTP server listens on | `5000` |
| `MONGO_URI` | MongoDB connection connection string | `mongodb://localhost:27017/secure_connect_db` |
| `JWT_SECRET` | Secret key used to sign and verify JWT tokens | `supersecretjwtkey123` |

### Running the Server

- **Development Mode (with auto-reload via Nodemon):**
  ```bash
  npm run dev
  ```

- **Production Mode:**
  ```bash
  npm start
  ```

Upon successful startup, the console will output:
```
MongoDB Connected Successfully
Server running on port 5000
```

---

## API Documentation

All request bodies must be sent as `application/json`. Authenticated routes require an `Authorization` header formatted as:
```
Authorization: Bearer <your_jwt_token>
```

### Authentication Endpoints

#### 1. Register User
- **URL:** `POST /api/auth/signup`
- **Access:** Public
- **Request Body:**
  ```json
  {
    "username": "johndoe",
    "email": "john@example.com",
    "password": "secretpassword"
  }
  ```
- **Responses:**
  - `201 Created`: User successfully registered.
    ```json
    {
      "message": "User registered",
      "id": "67352345abcdef0123456789",
      "username": "johndoe",
      "email": "john@example.com"
    }
    ```
  - `400 Bad Request`: Missing fields or password shorter than 6 characters.
  - `409 Conflict`: User with this email already exists.

#### 2. Login User
- **URL:** `POST /api/auth/login`
- **Access:** Public
- **Request Body:**
  ```json
  {
    "email": "john@example.com",
    "password": "secretpassword"
  }
  ```
- **Responses:**
  - `200 OK`: Login successful, returns JWT token.
    ```json
    {
      "message": "Login successful",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
    ```
  - `401 Unauthorized`: Invalid email or password.

---

### Notes Endpoints

All notes endpoints require the user to be authenticated via Bearer token (`authMiddleware`).

#### 1. Create a Note
- **URL:** `POST /api/notes`
- **Access:** Authenticated Users
- **Request Body:**
  ```json
  {
    "title": "Meeting Notes",
    "content": "Discuss project sprint deliverables."
  }
  ```
- **Response `201 Created`:**
  ```json
  {
    "_id": "67352489abcdef0123456790",
    "title": "Meeting Notes",
    "content": "Discuss project sprint deliverables.",
    "owner": "67352345abcdef0123456789",
    "createdAt": "2026-11-13T17:50:00.000Z",
    "updatedAt": "2026-11-13T17:50:00.000Z",
    "__v": 0
  }
  ```

#### 2. Get User's Notes
- **URL:** `GET /api/notes`
- **Access:** Authenticated Users
- **Response `200 OK`:** Array of notes owned by the authenticated user.

#### 3. Get Note by ID
- **URL:** `GET /api/notes/:id`
- **Access:** Authenticated Users (owner only)
- **Responses:**
  - `200 OK`: Note object.
  - `404 Not Found`: Note does not exist or does not belong to the user.
  - `400 Bad Request`: Invalid Note ID format.

#### 4. Update Note
- **URL:** `PUT /api/notes/:id`
- **Access:** Authenticated Users (owner only)
- **Request Body:**
  ```json
  {
    "title": "Updated Meeting Notes",
    "content": "Updated deliverables and deadlines."
  }
  ```
- **Responses:**
  - `200 OK`: Updated note object.
  - `404 Not Found`: Note does not exist or does not belong to the user.

#### 5. Delete Note
- **URL:** `DELETE /api/notes/:id`
- **Access:** Authenticated Users (owner only)
- **Response `200 OK`:**
  ```json
  {
    "message": "Note deleted successfully"
  }
  ```

#### 6. Admin: Get All Notes
- **URL:** `GET /api/notes/admin/all`
- **Access:** Admin Only (`role: "admin"`)
- **Responses:**
  - `200 OK`: Returns all notes across all users with populated owner details (`username`, `email`).
  - `403 Forbidden`: `Access denied. Admin only.`

---

### Health Check

- **URL:** `GET /`
- **Access:** Public
- **Response `200 OK`:** `"Secure Connect Server is running"`

---

## Real-Time Communication (Socket.IO)

The server exposes a Socket.IO endpoint on the same server instance with CORS enabled.

### Authentication Handshake

Clients must pass their JWT token in `auth.token` during the connection handshake:

```javascript
import { io } from "socket.io-client";

const socket = io("http://localhost:5000", {
  auth: {
    token: "<YOUR_JWT_TOKEN>"
  }
});

socket.on("connect", () => {
  console.log("Connected with socket ID:", socket.id);
});

socket.on("connect_error", (err) => {
  console.error("Connection failed:", err.message); // e.g. "Authentication token required" or "Invalid token"
});
```

### Client Events

#### Join Room (`joinRoom`)
Join an isolated chat room:
```javascript
socket.emit("joinRoom", "project-discussions");
```

#### Send Message (`chatMessage`)
Send a message to the currently joined room:
```javascript
socket.emit("chatMessage", "Hello team!");
```

### Server Broadcasts

#### Receive Message (`chatMessage`)
Listen for messages broadcasted to the room:
```javascript
socket.on("chatMessage", (data) => {
  console.log(`[${data.room}] ${data.username}: ${data.message} (${data.time})`);
});
```

---

## Event-Driven Architecture

The backend leverages a Node.js `EventEmitter` attached to the Express app context (`app.get("eventEmitter")`). It emits events whenever notes undergo mutations:

| Event | Trigger | Payload |
| :--- | :--- | :--- |
| `noteCreated` | A new note is successfully created via `POST /api/notes` | Created note object |
| `noteUpdated` | A note is modified via `PUT /api/notes/:id` | Updated note object |
| `noteDeleted` | A note is deleted via `DELETE /api/notes/:id` | Deleted note object |

These events can be extended to support real-time notifications, auditing, metrics, or third-party webhooks without coupling business logic.

---

## Testing

The API endpoints and authentication flows have been verified using [Thunder Client](https://www.thunderclient.com/). Visual proof of testing is available in the [`WEBDEV_lab4/test on thunderclient/`](./WEBDEV_lab4/test%20on%20thunderclient/) directory, covering:
- User signup and duplicate user handling
- User login and token generation
- Authorized note creation and retrieval
- Route protection against unauthenticated requests
- Note modification and deletion validation
- Admin authorization enforcement (`/api/notes/admin/all`)

