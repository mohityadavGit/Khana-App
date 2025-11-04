---


# Khana App

This project is a full-stack application with **Next.js frontend**, **Node.js/Express backend**, **PostgreSQL database**, and **Prisma ORM**, all running inside Docker containers.

---
---

## Prerequisites

- Docker & Docker Compose installed
- Node.js (optional, for local testing)
- PostgreSQL client (optional, for inspecting DB)

---

## Project Structure

```

khna-app/
│
├─ backend/
│   ├─ src/               # Node.js backend code
│   ├─ prisma/
│   │   └─ schema.prisma  # Prisma schema file
│   ├─ package.json
│   ├─ Dockerfile.dev
│   └─ .env.docker
│
├─ frontend/
│   ├─ src/               # React frontend code
│   ├─ package.json
│   └─ Dockerfile.dev
│
├─ images/
│   └─ architecture.png   
│
├─ docker-compose.yml
└─ README.md

````
## Environment Variables
---



Backend uses `.env.docker` for Docker:

```env
DATABASE_URL=postgresql://khanauser:khanapass@db:5432/khanadb
JWT_SECRET=your_jwt_secret
PORT=5000
# other environment variables
````

> Make sure this matches the service name `db` in `docker-compose.yml`.

---

## Docker Setup

Docker Compose file (`docker-compose.yml`) defines 3 services:

1. **backend** - Node.js/Express app
2. **frontend** - React app
3. **db** - PostgreSQL database

Port mapping:

* Frontend: `localhost:3000 -> container:3000`
* Backend: `localhost:5000 -> container:5000`
* Database: internal container port `5432`

---

## Running the Application

### 1. Build and start containers:

```bash
docker compose up --build
```

> ⚠️ Do **not** run `prisma migrate dev` during the Docker build. The database must be ready before migrations.

### 2. Run Prisma migrations:

Open a new terminal and run:

```bash
docker compose exec backend npx prisma migrate dev --name init
```

* This creates database tables according to your Prisma schema.
* Prisma client will be generated automatically if not already.

### 3. Verify containers:

```bash
docker compose ps
```

Expected output:

```
NAME                  IMAGE                STATUS         PORTS
khana-app-backend-1   khana-app-backend    Up             0.0.0.0:5000->5000/tcp
khana-frontend        khana-app-frontend   Up             0.0.0.0:3000->3000/tcp
khana-app-db-1        postgres:15          Up             5432/tcp
```

---

## Prisma Migrations

* Prisma tracks applied migrations.
* Once migration is done, backend can be restarted freely.
* For schema changes:

```bash
docker compose exec backend npx prisma migrate dev --name your_migration_name
```

---

## Architecture Diagram

The application has three main components:

```
Frontend (React)
       │
       ▼
Backend (Node.js/Express + Prisma)
       │
       ▼
Database (PostgreSQL)
```
---

## Useful Commands

| Command                                                            | Description                         |
| ------------------------------------------------------------------ | ----------------------------------- |
| `docker compose up --build`                                        | Build & start all containers        |
| `docker compose up`                                                | Start containers without rebuilding |
| `docker compose down`                                              | Stop & remove containers            |
| `docker compose exec backend bash`                                 | Enter backend container shell       |
| `docker compose exec backend npx prisma generate`                  | Generate Prisma client              |
| `docker compose exec backend npx prisma migrate dev --name <name>` | Run migrations                      |
| `docker compose logs -f backend`                                   | View backend logs live              |

---

## Notes

* Always ensure `.env.docker` matches your database service name (`db`).
* Nodemon in backend auto-reloads server on file changes.
* Frontend uses port 3000 and hot reloads.
* Docker volumes ensure database persists even if container restarts.
