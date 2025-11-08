# QuickFix_HUB

QuickFix_HUB is a marketplace web application that connects consumers with local service providers (e.g., electricians, plumbers, handymen). The project is split into a React + Vite frontend and a Node/Express backend using MongoDB (Mongoose). Firebase is used for authentication and optional storage integration.

This README documents the project structure, how to run it locally (dev container / Ubuntu 24.04), environment variables, useful commands, deployment notes, testing recommendations, and security/operational considerations.

---

## Table of contents

- Project overview
- Key features
- Repo structure
- Prerequisites
- Environment variables
- Local development (backend + frontend)
- Seeding the database
- Build & production preview
- Docker (recommended approach)
- Testing & quality
- Deployment & CI/CD recommendations
- Security & operational notes
- Contributing
- License

---

## Project overview

- Frontend: React (Vite) + TailwindCSS, organized by feature pages and components.
  - Entry: `frontend/main.jsx`, `frontend/App.jsx`
  - Components: `frontend/src/components/*`
  - Pages: `frontend/src/Pages/*`
  - Firebase client config: `frontend/src/utils/Firebase.js`
- Backend: Node + Express, REST API with route → controller → model pattern.
  - Entry: `backend/index.js`
  - DB connection: `backend/db.js`
  - Models: `backend/model/*` (User, ServiceProvider, Service, Booking, Review, AdminModel)
  - Routes: `backend/routes/*`
  - Firebase token verification helper: `backend/firebase.js`
  - Local seed data: `backend/seed.js`

---

## Key features

- User & provider signup / login (Firebase-auth)
- Provider profiles and service listings
- Search & filter providers
- Booking workflow (users book providers)
- Reviews & ratings
- Admin panel (user/provider/booking management)
- Map view for provider locations (map component)
- Toast notifications and visual charts for stats

---

## Repo structure (top-level)

- `/frontend` — React app (Vite)
- `/backend` — Express API, models, routes, controllers
- `/package.json` — workspace-level dependencies (dev-only; primary deps live in each package)
- `/package-lock.json`, `/node_modules` — lock & installed packages (workspace-wide)

Refer to the `frontend/README.md` for any frontend-specific notes.

---

## Prerequisites

- Node.js (LTS recommended; use the same Node version as CI)
- npm (or pnpm/yarn based on your preference)
- MongoDB instance (local, Docker, or cloud-managed like Atlas)
- Firebase project for auth (optional but recommended)
- (Optional) Docker & Docker Compose for containerized dev

Running inside the provided dev container: the environment is Ubuntu 24.04.2 LTS, common CLI tools are available (see container README or devcontainer config).

---

## Required environment variables

Create `.env` files in `backend/` and `frontend/` as needed. Typical vars:

Backend (`backend/.env`)
- PORT=5000
- MONGO_URI=<mongodb-connection-string>
- FIREBASE_PROJECT_ID=<firebase-project-id>
- FIREBASE_CLIENT_EMAIL=<firebase-client-email>
- FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
- (Optional) ADMIN_SECRET or other keys used by admin verification

Frontend (`frontend/.env`)
- VITE_FIREBASE_API_KEY=
- VITE_FIREBASE_AUTH_DOMAIN=
- VITE_FIREBASE_PROJECT_ID=
- VITE_FIREBASE_STORAGE_BUCKET=
- VITE_FIREBASE_MESSAGING_SENDER_ID=
- VITE_FIREBASE_APP_ID=
- VITE_FIREBASE_MEASUREMENT_ID=

Note: Vite uses `VITE_` prefixed env vars for browser exposure. Do not commit env files to source control.

---

## Local development

Open two terminals (or use a process manager / tmux).

1. Backend
```bash
cd backend
npm install
# copy or create .env with the variables above
npm start
# or if package.json uses dev script:
# npm run dev
```
- Backend default port is commonly `5000` (check `backend/index.js`).
- Ensure `MONGO_URI` is reachable before starting.

2. Frontend
```bash
cd frontend
npm install
# create frontend/.env with VITE_FIREBASE_*
npm run dev
```
- Vite dev server typically starts on `http://localhost:5173` (check output).

---

## Database seeding

To populate sample data for local testing:
```bash
cd backend
node seed.js
```
- `seed.js` creates sample users, providers, services, bookings and reviews. Use only in dev or staging environments.

---

## Build & production preview

1. Build frontend:
```bash
cd frontend
npm run build
```
2. Serve static `dist/` with a static server for preview:
```bash
npx serve dist
```
3. Integrate built frontend into Express static hosting by adding `express.static` to `backend/index.js` and pointing to the `dist` folder.

---

## Docker (recommended)

Suggested approach:
- Create Dockerfile for backend (Node base), multi-stage Dockerfile for frontend (build -> serve).
- Use `docker-compose.yml` to wire backend, frontend (optional), and MongoDB for local integration.

Example high-level steps:
```bash
# Build images
docker build -t quickfix-backend ./backend
docker build -t quickfix-frontend ./frontend
# Start services via docker-compose up
```

---

## Testing & quality

- This repository contains no formal test suite. Recommended tooling:
  - Backend: Jest + Supertest for route/controller tests; use `mongodb-memory-server` to isolate DB.
  - Frontend: Jest + React Testing Library + Playwright/Cypress for end-to-end flows.
- Linting: configure ESLint/Prettier in frontend and backend to maintain code style.

---

## Deployment & CI/CD (recommendations)

- Frontend: deploy Vite build to Vercel/Netlify, or S3 + CloudFront.
- Backend: containerize and deploy to ECS/EKS/GCP Cloud Run/Heroku. Use managed MongoDB (Atlas) for production.
- CI/CD pipeline example:
  - On PR: lint → unit tests → build
  - On main: build → integration tests → push docker image → deploy
- Use secret management (GitHub Secrets, AWS Secrets Manager) for env vars.

---

## Security & operational considerations

- Authentication: Firebase currently used for auth; backend verifies Firebase tokens (`backend/firebase.js`) — ensure token verification and role checks are tested.
- Store secrets securely; never commit `.env` or private keys.
- Harden API: add rate limiting, Helmet, input validation (Joi/Zod), request size limits.
- Protect file uploads: use signed URLs and storage rules if using Firebase Storage.
- Data privacy: add data deletion / export endpoints for GDPR compliance.

---

## Performance & scaling

- Add pagination and indexes for search endpoints.
- Cache frequent search responses (Redis) and use CDN for assets.
- Make backend stateless and use a queue (BullMQ / Redis) for background jobs (email, heavy processing).
- Use horizontal scaling behind a load balancer and a managed DB with replicas.

---

## Contributing

1. Fork the repo and create a feature branch.
2. Follow coding standards and update/add tests for new functionality.
3. Submit a PR with a clear description and testing steps.

---

## Troubleshooting (common issues)

- Backend fails to connect to MongoDB: verify `MONGO_URI` and network access.
- Firebase auth issues: confirm service account keys and client config match the same Firebase project.
- CORS errors: confirm backend CORS config and Vite proxy settings if used.
- Port conflicts: adjust backend `PORT` or Vite dev server port.

---

## Next steps / Roadmap (suggested)

- Integrate a payments provider (Stripe) for bookings.
- Add real-time updates for bookings (WebSockets).
- Harden RBAC and admin tooling.
- Add CI tests and production-grade monitoring (Sentry, Prometheus).

---

## Contact & license

- Add your preferred contact/maintainer info here.
- Choose and add a license file (e.g., MIT) to the repo.
