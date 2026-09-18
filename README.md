# ISPM ERP

ISPM ERP is a school management system designed to manage academic years, departments, students, payment schedules, payments, documents, and application settings from a single interface.

## Tech stack

- Frontend: React, Vite, TypeScript
- Backend: Express, Prisma, SQLite

## Project structure

```text
ispm-erp/
├── src/                 # Frontend application
├── public/              # Frontend static assets
├── server/              # Backend API, Prisma schema, seed data
│   ├── prisma/
│   └── src/
├── docs/                # Project documentation
└── package.json         # Frontend scripts and dependencies
```

## Prerequisites

- Node.js 20 or later
- npm
- Git

Windows paths used by the current project owner:

- Node.js: `C:\Users\hp\AppData\Local\Programs\node-v20.18.0-win-x64`
- Git: `C:\Program Files\Git\bin\git.exe`

## Local installation

### 1. Clone the repository

```bash
git clone https://github.com/yrachdy/ispm-erp.git
cd ispm-erp
```

### 2. Install frontend dependencies

```bash
npm install
```

### 3. Install backend dependencies

```bash
cd server
npm install
```

### 4. Configure environment variables

Create the frontend environment file:

```bash
cp .env.example .env
```

Create the backend environment file:

```bash
cd server
cp .env.example .env
```

### 5. Initialize the database

From `server/`:

```bash
npx prisma migrate deploy
npx prisma db seed
```

For local development, if you need to create the SQLite database from migrations:

```bash
npx prisma migrate dev
npx prisma db seed
```

### 6. Start the backend

From `server/`:

```bash
npm run dev
```

The API runs by default on `http://localhost:3001`.

### 7. Start the frontend

From the project root in a second terminal:

```bash
npm run dev
```

The frontend runs by default on `http://localhost:5173`.

## Default credentials

- Email: `admin@ispm.ma`
- Password: `admin123`

## Available features

- Authentication with JWT-based access control
- Academic year management
- Department management
- Student management
- Payment schedule management
- Payment tracking
- Document management
- Application settings management
- Dashboard and reporting views

## Development scripts

### Frontend

- `npm run dev` — start the Vite development server
- `npm run build` — build the frontend for production
- `npm run preview` — preview the production build locally

### Backend

From `server/`:

- `npm run dev` — start the Express API in development mode
- `npm run build` — compile the backend TypeScript code
- `npm run start` — run the compiled backend
- `npx prisma studio` — inspect the database

## Database

The backend uses Prisma with SQLite by default. See the database guide for model details, reset instructions, seeding, and production migration notes:

- [Database documentation](docs/DATABASE.md)

## Deployment

For a complete standalone deployment on a Linux VPS with Nginx, PM2, SSL, and update procedures, see:

- [Deployment guide](docs/DEPLOYMENT_VPS.md)