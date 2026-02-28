---
layout: default
title: Getting Started
nav_order: 1
parent: Developer Guide
---

# Getting Started

This guide covers the minimum steps to clone, configure, and run the Stakeholder Agent System locally.

## Prerequisites

Before you begin, ensure you have the required tools installed:

| Tool       | Version       | Purpose                 |
|------------|---------------|-------------------------|
| Node.js    | v20+          | Frontend runtime        |
| Python     | 3.11+         | Backend runtime         |
| Docker     | Latest stable | Containerization        |
| PostgreSQL | 15+           | Database                |
| Git        | Latest        | Version control         |

For complete development environment details, see [Development Workflows]({% link docs/development-workflows.md %}).

---

## Clone the Repositories

The project consists of separate frontend and backend repositories:

```bash
# Frontend application
git clone https://github.com/project-steak-holder/ai-frontend.git

# Backend AI service
git clone https://github.com/project-steak-holder/ai-backend.git
```

---

## Backend Setup

1. **Navigate to the backend directory:**

   ```bash
   cd ai-backend
   ```

2. **Create and activate a virtual environment:**

   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**

   ```bash
   curl https://uvm.astral.sh/install.sh | sh
   uv sync
   ```

4. **Configure environment variables:**

   Create a `.env` file with the required configuration:

   ```bash
   DATABASE_URL=postgresql://user:password@localhost:5432/stakeholder_db
   NEON_AUTH_SECRET=your_auth_secret
   AI_PROVIDER_API_KEY=your_api_key
   ```

5. **Run database migrations:**

   ```bash
   alembic upgrade head
   ```

6. **Start the backend service:**

   ```bash
   uvicorn app.main:app --reload
   ```

   The API will be available at `http://localhost:8000`.

---

## Frontend Setup

1. **Navigate to the frontend directory:**

   ```bash
   cd ai-frontend
   ```

2. **Install dependencies:**

   ```bash
   npm install
   ```

3. **Configure environment variables:**

   Create a `.env` file:

   ```bash
   VITE_API_URL=http://localhost:8000
   VITE_AUTH_DOMAIN=your_auth_domain
   ```

4. **Start the development server:**

   ```bash
   npm run dev
   ```

   The application will be available at `http://localhost:5173`.

---

## Verify Installation

1. Open your browser to `http://localhost:5173`
2. Create a new account or log in
3. Start a new conversation to verify the system is working

---

## Next Steps

Once you have the system running locally:

| Goal | Documentation |
|------|---------------|
| Understand the system design | [Architecture]({% link docs/architecture.md %}) |
| Learn about the tech stack | [Technology Stack]({% link docs/technology-stack.md %}) |
| Start contributing code | [Development Workflows]({% link docs/development-workflows.md %}) |
| Work on a feature | [Implementation Guidance]({% link docs/implementation-guidance.md %}) |
| Write tests | [Testing Strategy]({% link docs/testing-strategy.md %}) |
| Use the API | [API Documentation]({% link docs/api-documentation.md %}) |
| Troubleshoot issues | [Troubleshooting]({% link docs/troubleshooting.md %}) |
| Contribute to the project | [Contributing]({% link docs/contributing.md %}) |
