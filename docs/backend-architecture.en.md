# Indexu Core – Backend Architecture

> 📂 This document provides a **high-level overview** of the backend architecture of the Indexu Core project.  
> It does **not** include secrets, credentials, or private internal logic.

---

## 1. Overview

The **Indexu Core backend** is a Node.js + TypeScript API designed to:

- manage users and dimensional profiles;
- process and analyze media (videos);
- orchestrate AI engines (dimensions, reputation, participation, etc.);
- enable future Web3 integration and tokenization layers.

The architecture follows a **modular domain-oriented structure**:

- each domain (e.g. `profile`, `media`, `userdimensions`) is isolated as a module;
- controllers, services, routes, and DTOs are separated by responsibilities;
- heavy tasks run asynchronously using queues and workers.

---

## 2. Tech Stack (High Level)

- **Language:** Node.js + TypeScript  
- **HTTP Framework:** Express  
- **ORM:** Prisma  
- **Database:** PostgreSQL  
- **Queues & Jobs:** BullMQ + Redis  
- **Media Storage:** S3-compatible Object Storage  
- **Authentication:** JWT + session middleware  
- **Architecture Style:** modular, domain-oriented

---

## 3. Folder Structure (Simplified)

```bash
src/
  app.ts              # App initialization
  server.ts           # HTTP server bootstrap
  typings.ts          # Global typing

  auth/               # Authentication logic
  config/             # Redis, queues, storage configs
  db/                 # Prisma client
  middlewares/        # Auth & utility middlewares

  modules/            # Domain modules
    profile/
    media/
    userdimensions/
    networkai/
    feed/
    reputation/
    participation/
    tokenengine/
    userai/
    web3/

  onboarding/         # Onboarding flow
  upload/             # Upload & storage integration
  user/               # User controllers
  workers/            # Video processing jobs
  utils/              # JWT, hash, storage helpers
  types/              # Shared typings
Note: this structure is intentionally generic and does not expose private implementation details.

4. Request Flow (HTTP API)
The client (web/app) sends an HTTP request.

Request may pass through logging/auth middleware.

Controller receives the request.

Controller calls a service where business logic lives.

Service interacts with:

Prisma (database)

job queues

media storage

other modules

Response returns as JSON.

5. Media Pipeline (High-Level)
Client requests a video upload.

Backend generates an upload URL + creates a media entry.

Upload completes → queue triggers processing job.

Worker handles:

video processing

frame extraction

metadata update

AI analysis (future stage) runs separately.

Media ends as ready or failed.

No internal buckets, URLs, endpoints or secrets are revealed here.

6. Prisma Models (Simplified)
ts
Copiar código
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  medias Media[]
}

model Media {
  id        String      @id @default(cuid())
  userId    String
  type      String
  status    MediaStatus @default(uploading)
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt

  user   User          @relation(fields: [userId], references: [id])
  frames MediaFrame[]
}

enum MediaStatus {
  uploading
  processing
  ready
  failed
  deleted
}
7. Security & Best Practices
.env is not committed to GitHub

no secrets or storage URLs are exposed

environment variables configure dependencies

DB, storage and external services are decoupled

Example:

bash
Copiar código
# .env (not versioned)
DATABASE_URL="postgres://user:password@host:5432/db"
REDIS_URL="redis://host:6379"
STORAGE_ENDPOINT="https://provider"
8. Technical Roadmap
Consolidate media analysis pipeline (AI multimodal)

Expand dimensional engine (IND / EXD / U)

Connect reputation system to feed ranking

Token engine & web3 integration (future phase)

Improve public documentation gradually

9. License Notice
This project is under active development.
Architecture and modules may evolve over time.


