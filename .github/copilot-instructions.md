<!-- Brief, action-oriented instructions for AI coding agents working on this repo -->
# Copilot instructions — Blood-Bank-MERN

These instructions orient an AI coding agent to the structure, conventions, and key workflows of this repository so it can make safe, minimal, and correct code changes.

**Big Picture**
- **Stack:** MERN (MongoDB, Express, React, Node). Backend lives at the repo root; frontend lives in `client/`.
- **API namespace:** All server endpoints are mounted under `/api/v1/*` (see `server.js`).
- **Design:** Classic MVC on backend: `controllers/`, `routes/`, `models/`, `middleware/`. Frontend is React + Redux Toolkit under `client/src/`.

**Run / build commands (precise)**
- Install backend deps: `npm install` (at repo root).
- Install client deps: `npm run client-install` (or `cd client && npm install`).
- Run both concurrently: `npm start` (root) — uses `concurrently` to run `nodemon server.js` and React dev server.
- Or run just backend: `npm run server`; just client: `npm run client`.

**Environment & integration details**
- Backend env vars expected: `JWT_SECRET`, `MONGO_URL`, `PORT` (defaults to 8080). See `config/db.js` and `server.js`.
- Frontend env: `REACT_APP_BASEURL` — README suggests `http://localhost:8080/api/v1`. The frontend `client/src/services/API.js` uses this value as the axios baseURL.
- Authentication: JWT bearer tokens. Backend middleware `middleware/authMiddleware.js` expects an `Authorization` header in the form `Bearer <token>` and extracts `userId` into `req.body.userId`.

**Code patterns and gotchas (use these examples)**
- Centralized API client: `client/src/services/API.js` creates an axios instance and attaches `Authorization: Bearer <token>` when `localStorage.token` exists. Prefer using that instance for frontend HTTP calls.
- Frontend dispatch pattern: local helper functions (e.g. `client/src/services/authService.js`) call `store.dispatch(...)` directly with `userLogin` / `userRegister` actions—follow this pattern for auth flow changes.
- Role values: user roles are defined in `models/userModel.js` enum: `"admin", "organisation", "donar", "hospital"`. The frontend must use these exact strings (note the project uses `donar` not `donor`).
- Current-user endpoint: `GET /api/v1/auth/current-user` is protected by `authMiddleware` — send `Authorization: Bearer <token>`.

**File locations to reference when making changes**
- Entry point: `server.js` (mounts routes and middleware).
- Auth flow: `controllers/authController.js`, `routes/authRoute.js`, `middleware/authMiddleware.js`.
- Models: `models/` (e.g. `userModel.js`, `inventoryModel.js`).
- Frontend API wrapper: `client/src/services/API.js` and `client/src/services/authService.js`.
- Redux: `client/src/redux/features/auth/` contains `authActions.js` and `authSlice.js` — prefer existing actions when wiring UI to backend.

**Editing rules for AI agents**
- Be conservative: prefer small, local changes that preserve existing patterns (e.g., dispatching actions, using `API` instance, honoring role strings).
- When changing authentication or headers, update both `client/src/services/API.js` and any middleware that parses the header (`middleware/authMiddleware.js`).
- Avoid renaming role strings or other canonical identifiers without a coordinated repo-wide change and tests—these are used in DB validation and throughout code.
- Keep API paths under `/api/v1/` unless adding a new versioned route.

**Testing & verification steps to include with PRs**
1. Run `npm run client-install` and `npm install` at root, then `npm start`. Confirm backend logs `Server running on port 8080` and React starts on `:3000`.
2. Register a user via the UI or `POST /api/v1/auth/register`. Then `POST /api/v1/auth/login` to obtain a token.
3. Call `GET /api/v1/auth/current-user` with `Authorization: Bearer <token>` to verify middleware + controller flow (this populates `req.body.userId`).

**When unsure or blocked**
- If a change touches auth, database schemas, or role names, request an explicit review from a human — these are high-risk, cross-cutting areas.
- If environment variables are unclear in a PR, add a short note describing the expected `.env` keys and recommended values (see README for intended defaults).

---
If anything is missing or you'd like this trimmed/expanded (examples, more endpoints, or testing scripts), tell me which area to focus on and I'll refine it.
