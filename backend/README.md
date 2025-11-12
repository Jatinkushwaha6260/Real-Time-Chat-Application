# Real Time Chat Application (Production-ready README)

Short description
- Real-time chat backend built with Node.js, Express, MongoDB (Mongoose) and Socket.IO.
- Features: user register, login, logout, get other users; message controllers: send message, get messages.
- Security: JWT for auth, bcrypt for password hashing, cookie-parser for cookies, CORS handling, dotenv for config.
- Dev tooling: nodemon for development.

Table of contents
- Features
- Tech stack
- Quick start (development & production)
- Environment variables
- API endpoints (HTTP)
- Socket.IO events
- Security & production considerations
- Deployment tips
- Troubleshooting

Features
- User registration (email/username + password, hashed with bcrypt)
- Login (JWT issued, stored as HttpOnly cookie or returned in response)
- Logout (invalidate cookie / client removes token)
- Get other users (authenticated)
- Messages: send message (persisted to MongoDB), get conversation messages (paginated)
- Real-time: Socket.IO for live messaging and presence

Tech stack
- Node.js + Express
- MongoDB + Mongoose
- Socket.IO
- jsonwebtoken (JWT)
- bcryptjs
- cookie-parser
- cors
- dotenv
- nodemon (dev)

Quick start

1) Clone repo
- git clone <repository-url>
- cd backend

2) Install
- npm install

3) Environment
- Create a `.env` file based on the example below.

4) Development
- npm run dev
  - Uses nodemon to restart on changes.

5) Production
- Build step (if any) then:
- NODE_ENV=production pm2 start index.js --name chat-app
- or: npm start (ensure environment variables and process manager in place)

Environment variables (.env.example)
- PORT=4000
- MONGO_URI=mongodb+srv://USER:PASS@cluster.example.mongodb.net/dbname?retryWrites=true&w=majority
- JWT_SECRET=your_strong_jwt_secret_here
- JWT_EXPIRES_IN=7d
- COOKIE_NAME=token
- COOKIE_SECURE=true            # set true if using HTTPS
- CLIENT_URL=https://yourfrontend.example.com
- NODE_ENV=development

API endpoints (HTTP)
- All endpoints under /api (example)

Authentication
- POST /api/auth/register
  - Body: { "name": "string", "email": "string", "password": "string" }
  - Response: created user (no password) + token or set HttpOnly cookie

- POST /api/auth/login
  - Body: { "email": "string", "password": "string" }
  - Response: token (JWT) or sets cookie

- POST /api/auth/logout
  - Clears cookie or instructs client to remove token
  - Response: { "message": "Logged out" }

User
- GET /api/users
  - Auth required (JWT)
  - Returns list of other users (exclude requesting user)
  - Query params: ?limit=50&skip=0 (optional)

Messages
- POST /api/messages/send
  - Auth required
  - Body: { "to": "recipientUserId", "text": "string" }
  - Stores message in DB, emits Socket event to recipient
  - Response: saved message object

- GET /api/messages/:conversationId?limit=50&skip=0
  - Auth required
  - Returns paginated messages for a conversation (or messages between two user ids)

Authentication details
- Use JWT in Authorization header: Authorization: Bearer <token>
- Or store JWT as HttpOnly, Secure cookie named as COOKIE_NAME. The backend should accept both forms.
- Validate JWT on protected routes. Expire token and handle refresh (optional).

Socket.IO usage
- Server initialization:
  - const io = new Server(server, { cors: { origin: process.env.CLIENT_URL, credentials: true } });
- Events (recommended)
  - connect / disconnect
  - client -> server:
    - "user:connect" { userId }  // register socket id for user
    - "message:send" { to, text, tempId? }
  - server -> client:
    - "message:received" { message }
    - "user:online" { userId }
    - "user:offline" { userId }
- On server message receive: persist to DB, then emit to recipient by socketId(s). Fallback: store unread and deliver when recipient reconnects.

Data models (high-level)
- User: { name, email (unique), passwordHash, lastSeen, avatar?, sockets: [] }
- Message: { from, to, text, createdAt, delivered: bool, read: bool, conversationId }

Security & production considerations
- Use HTTPS; set cookie Secure = true and SameSite as needed.
- Store JWT secret in environment variables and rotate periodically.
- Hash passwords using bcrypt with an appropriate salt rounds (e.g., 10-12).
- Rate-limit auth endpoints to prevent brute force.
- Validate & sanitize all inputs (express-validator or Joi).
- Use Helmet to set secure HTTP headers.
- Enable CORS only for trusted origins (CLIENT_URL).
- Avoid exposing stack traces in production.
- Use strong CSP and XSS protections on frontend.
- Limit file uploads and validate content types if present.

Logging & Monitoring
- Use structured logging (winston/pino).
- Monitor with services (Sentry, LogDNA) and track socket errors separately.
- Add health-check endpoint: GET /api/health

