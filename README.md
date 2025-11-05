
# Khana App

This project is a full-stack application with **Next.js frontend**, **Node.js/Express backend**, **PostgreSQL database**, and **Prisma ORM**, all running inside Docker containers.

---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Project Structure](#project-structure)  
3. [Environment Variables](#environment-variables)  
4. [Docker Setup](#docker-setup)  
5. [Running the Application](#running-the-application)  
6. [Prisma Migrations](#prisma-migrations)  
7. [Pulling from Docker Hub](#pulling-from-docker-hub)  
8. [Useful Commands](#useful-commands)  
9. [Notes](#notes)  
10. [Architecture Diagram](#architecture-diagram)  

---

## Prerequisites

* Docker & Docker Compose installed  
* Node.js (optional, for local testing)  
* PostgreSQL client (optional, for inspecting DB)  

---

## Project Structure

```

khana-app/
│
├─ backend/
│   ├─ src/
│   ├─ prisma/
│   │   └─ schema.prisma
│   ├─ package.json
│   ├─ Dockerfile.dev
│   └─ .env.docker
│
├─ frontend/
│   ├─ src/
│   ├─ package.json
│   └─ Dockerfile.dev
│
├─ docker-compose.yml
└─ README.md

````

---

## Environment Variables

Backend uses `.env.docker` for Docker:

```env
DATABASE_URL="postgresql://khanauser:khanapass@db:5432/khanadb?schema=public"
PORT=5000
JWT_SECRET=your_jwt_secret
````

> Make sure this matches the service name `db` in `docker-compose.yml`.

---

## Docker Setup

Docker Compose file (`docker-compose.yml`) defines 3 services:

1. **db** - PostgreSQL database
2. **backend** - Node.js/Express app
3. **frontend** - React app

**Port mapping:**

* Frontend: `localhost:3000 -> container:3000`
* Backend: `localhost:5000 -> container:5000`

---

## Running the Application

### Option 1: Build locally

```bash
docker compose up --build
```

> ⚠️ Important: **Do not run `prisma migrate dev` during Docker build**. Run it after containers are up.

Then, run Prisma migrations:

```bash
docker compose exec backend npx prisma migrate dev --name init
```

---

### Option 2: Pull images from Docker Hub (No local build needed)

1. Pull frontend & backend images:

```bash
docker pull mohityadavdocker/khana-app-frontend:v1
docker pull mohityadavdocker/khana-app-backend:v1
```

2. Update `docker-compose.yml` to use these images:

```yaml
backend:
  image: mohityadavdocker/khana-app-backend:v1
  environment:
    DATABASE_URL: postgres://khanauser:khanapass@db:5432/khanadb
    PORT: 5000
  depends_on:
    - db
  ports:
    - "5000:5000"
  networks:
    - khananet

frontend:
  image: mohityadavdocker/khana-app-frontend:v1
  container_name: khana-frontend
  ports:
    - "3000:3000"
  depends_on:
    - backend
  networks:
    - khananet
```

3. Start containers:

```bash
docker compose up
```

> Then run Prisma migrations if database schema is not ready:

```bash
docker compose exec backend npx prisma migrate dev --name init
```

---

## Prisma Migrations

* Prisma tracks migrations.
* After migration, backend can be restarted freely.
* If schema changes:

```bash
docker compose exec backend npx prisma migrate dev --name <your_migration_name>
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

* Ensure `.env.docker` matches the database service name (`db`).
* Nodemon in backend auto-reloads server on changes.
* Frontend uses port 3000 and hot reloads.
* Docker volumes ensure database persists even if container restarts.

---

## Architecture Diagram

```
+-------------------+        +-------------------+        +------------------+
|   Frontend (3000) | <----> | Backend (5000)    | <----> | PostgreSQL (5432)|
+-------------------+        +-------------------+        +------------------+
```

* Frontend communicates with Backend via REST API.
* Backend communicates with PostgreSQL using Prisma ORM.
* Docker network `khananet` connects all containers internally.
