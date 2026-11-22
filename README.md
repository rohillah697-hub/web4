# Secure Connect and Communication Platform

A secure, modular, and beginner-friendly backend platform built with Node.js, Express.js, MongoDB (official driver), bcrypt, JSON Web Tokens (JWT), and Socket.IO.

Designed specifically for college computer science curriculum and practical defense, emphasizing clear architectural separation, defensive coding practices, and zero unnecessary dependencies or boilerplate frameworks.

---

## 1. Project Purpose & Features

* **Custom Authentication Flow**: Public user registration and secure login built from scratch without Passport or 3rd-party auth frameworks.
* **Password Hashing**: Irreversible hashing using `bcrypt` with 12 salt rounds.
* **Stateless JWT Authorization**: Cryptographically signed access tokens issued on login, verified on protected routes via custom Express middleware.
* **Role-Based Access Control (RBAC)**: Enforces access restrictions between ordinary `user` and privileged `admin` roles, preventing privilege-escalation attacks.
* **Secure Notes API**: Complete CRUD operations with strict database-level owner isolation (`userId: req.user.userId`).
* **Real-Time Communication with Socket.IO**: JWT-authenticated WebSockets supporting authorized private conversation rooms (`direct:<userA>_<userB>`), message persistence, and prevention of sender impersonation.
* **Internal Event-Driven Architecture**: Decoupled process-level audit logging using Node.js built-in `EventEmitter`.
* **Zero Bloat**: Uses only the dependencies permitted by the project specification.

---

## 2. Prerequisites

* **Node.js**: v18.0.0 or higher (Tested and confirmed on Node.js v24.15.0).
* **npm**: v9.0.0 or higher (Tested on npm v11.17.0).
* **MongoDB**: A running local instance of MongoDB on default port `27017`.

To verify your local setup in Windows PowerShell:
```powershell
node -v
npm -v
Get-Service -Name *mongo*
```

---

## 3. Technology Stack & Dependencies

| Dependency | Purpose | Why Selected |
| :--- | :--- | :--- |
| **express** | HTTP REST API framework | Clean, minimal, standard for Node.js REST services. |
| **mongodb** | Official MongoDB driver | Direct, transparent MongoDB operations without Mongoose abstraction overhead. |
| **bcrypt** | Password hashing algorithm | Adaptive work factor (salt rounds = 12) resistant to brute-force and dictionary attacks. |
| **jsonwebtoken** | Token generation & verification | Industry-standard stateless identity tokens (RFC 7519). |
| **dotenv** | Environment configuration | Keeps secrets, ports, and connection strings out of source control. |
| **socket.io** | Real-time WebSocket server | Bi-directional communication for private conversation rooms. |
| **socket.io-client** | CLI test client | Allows testing real-time events without needing a frontend UI. |

---

## 4. Setup and Installation

### Step 1: Install Dependencies
In your project root:
```powershell
npm install
```

### Step 2: Environment Configuration
Copy the placeholder environment file to create your local `.env`:
```powershell
Copy-Item .env.example .env
```

Generate a secure, cryptographically random 64-character secret for `JWT_SECRET`:
```powershell
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Open `.env` and configure your settings:
```ini
PORT=3000
NODE_ENV=development
MONGO_URI=mongodb://127.0.0.1:27017/secure_connect_platform
JWT_SECRET=your_newly_generated_64_character_hex_secret_here
JWT_EXPIRES_IN=1h
BCRYPT_SALT_ROUNDS=12
```

> **Security Note:** Never commit `.env` to Git. It is already added to `.gitignore`.

---

## 5. Starting the Server

### Production / Standard Mode:
```powershell
npm start
```

### Development Mode (Auto-reloads on file change using Node.js native watch):
```powershell
npm run dev
```

Server starts at: `http://localhost:3000`  
Health check endpoint: `http://localhost:3000/api/health`

---

## 6. Local Admin Provisioning (Safe Classroom Demo)

Because public signups can never grant admin privileges, a dedicated CLI script is provided to safely provision an initial local administrator:

```powershell
npm run seed:admin
```
*(Creates or elevates `admin@secureconnect.local` with password `Admin@12345`).*

Or provision custom admin credentials:
```powershell
node scripts/createAdmin.js myadmin@college.edu MyPassword123 college_admin
```

---

## 7. API Summary

| Method | Endpoint | Auth Required | Role | Description |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/signup` | No | Public | Register new user account (`role: 'user'`). |
| `POST` | `/api/auth/login` | No | Public | Verify credentials, return signed JWT. |
| `GET` | `/api/auth/me` | Bearer JWT | Any | Get profile of logged-in user. |
| `POST` | `/api/notes` | Bearer JWT | Any | Create a new private note. |
| `GET` | `/api/notes` | Bearer JWT | Any | List notes belonging to caller. |
| `GET` | `/api/notes/:id` | Bearer JWT | Any | Get single note (owner enforced). |
| `PUT` | `/api/notes/:id` | Bearer JWT | Any | Update note (owner enforced). |
| `DELETE` | `/api/notes/:id` | Bearer JWT | Any | Delete note (owner enforced). |
| `GET` | `/api/admin/users` | Bearer JWT | `admin` | List all users (admin-only RBAC). |
| `GET` | `/api/health` | No | Public | Service health check. |

For detailed payloads and responses, consult [docs/API_GUIDE.md](file:///docs/API_GUIDE.md).

---

## 8. How to Test the Project

### Automated Test Runner (All 19 Scenarios)
With the server running in one terminal, run in a second terminal:
```powershell
npm test
```
This automatically verifies signup, duplicate checks, login, invalid credentials, token verification, token expiration, note CRUD, cross-user note isolation, RBAC denial/approval, Socket.IO authenticated connections, unauthorized socket rejection, room authorization, EventEmitter audit logging, and input validation.

### Socket.IO CLI Test Client
Test real-time messaging without a frontend:
```powershell
# 1. Login to get a token:
$res = Invoke-RestMethod -Uri "http://localhost:3000/api/auth/login" -Method Post -ContentType "application/json" -Body '{"email":"admin@secureconnect.local","password":"Admin@12345"}'

# 2. Run test client:
node scripts/testClient.js $res.data.token global:announcements "Testing live Socket.IO connection"
```

For complete manual testing instructions and PowerShell commands, see [docs/TESTING_CHECKLIST.md](file:///docs/TESTING_CHECKLIST.md).

---

## 9. Known Limitations & College Context

* **In-Memory EventEmitter**: Node.js `EventEmitter` operates solely within a single server process. In a distributed multi-server cluster, a message broker like Redis Pub/Sub or RabbitMQ would be needed.
* **Direct Room Chat Model**: The current conversation authorization model is optimized for deterministic 1-on-1 pairs (`direct:<userA>_<userB>`) and global announcements. Multi-user dynamic groups with invite/membership hierarchies can be added in future iterations.
* **Token Invalidation**: Stateless JWTs cannot be revoked immediately before expiry without an active token blocklist in a cache (like Redis). Tokens expire automatically after the configured duration (`JWT_EXPIRES_IN=1h`).
