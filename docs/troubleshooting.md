---
layout: default
title: Troubleshooting
nav_order: 10
---

# Troubleshooting

This guide covers common issues developers may encounter when working with the Stakeholder Agent System and provides solutions for resolving them.

---

## Environment Setup Issues

### Node.js Issues

**Problem: `node: command not found`**

Node.js is not installed or not in your PATH.

```bash
# Check if Node.js is installed
node --version

# Install Node.js v20+ using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
```

**Problem: `npm ERR! ERESOLVE unable to resolve dependency tree`**

Dependency conflicts between packages.

```bash
# Clear npm cache and reinstall
rm -rf node_modules package-lock.json
npm cache clean --force
npm install
```

**Problem: Node version mismatch**

The project requires Node.js v20+.

```bash
# Check current version
node --version

# Switch to correct version using nvm
nvm install 20
nvm use 20

# Set as default
nvm alias default 20
```

---

### Python Issues

**Problem: `python: command not found` or wrong Python version**

Python 3.11+ is not installed or not the default.

```bash
# Check Python version
python3 --version

# On Ubuntu/Debian
sudo apt update
sudo apt install python3.11 python3.11-venv

# On macOS with Homebrew
brew install python@3.11
```

**Problem: `pip install` fails with permission errors**

Installing packages globally without proper permissions.

```bash
# Always use a virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**Problem: `ModuleNotFoundError` after installing packages**

Virtual environment not activated or wrong environment.

```bash
# Verify virtual environment is active
which python  # Should show path within your venv directory

# Reactivate if needed
source venv/bin/activate
pip install -r requirements.txt
```

---

### Docker Issues

**Problem: `docker: command not found`**

Docker is not installed.

```bash
# On Ubuntu
sudo apt update
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

**Problem: `Got permission denied while trying to connect to the Docker daemon`**

User not in the docker group.

```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Apply changes without logging out
newgrp docker
```

**Problem: Docker containers fail to start**

Check container logs for specific errors.

```bash
# View container logs
docker logs <container_name>

# Check container status
docker ps -a

# Restart containers
docker-compose down
docker-compose up -d
```

---

## Database Connection Problems

### PostgreSQL Connection Issues

**Problem: `connection refused` to PostgreSQL**

Database server is not running or wrong connection details.

```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Start PostgreSQL
sudo systemctl start postgresql

# Verify connection details in .env
cat .env | grep DATABASE_URL
```

**Problem: `FATAL: password authentication failed`**

Incorrect database credentials.

```bash
# Verify your DATABASE_URL format
# Format: postgresql://user:password@host:port/database

# Test connection manually
psql postgresql://user:password@localhost:5432/stakeholder_db
```

**Problem: `FATAL: database "stakeholder_db" does not exist`**

Database has not been created.

```bash
# Connect as postgres user and create database
sudo -u postgres psql
CREATE DATABASE stakeholder_db;
CREATE USER your_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE stakeholder_db TO your_user;
\q
```

### Migration Issues

**Problem: `alembic upgrade head` fails**

Migration conflicts or database state issues.

```bash
# Check current migration status
alembic current

# View migration history
alembic history

# If database is out of sync, stamp to current
alembic stamp head

# Generate a new migration if schema changed
alembic revision --autogenerate -m "description"
```

**Problem: `Target database is not up to date`**

Pending migrations need to be applied.

```bash
# Apply all pending migrations
alembic upgrade head

# Or upgrade to specific revision
alembic upgrade <revision_id>
```

---

## API/Backend Issues

### Server Startup Issues

**Problem: `uvicorn` fails to start**

Port already in use or missing dependencies.

```bash
# Check if port 8000 is in use
lsof -i :8000

# Kill process using the port
kill -9 <PID>

# Start on a different port
uvicorn app.main:app --reload --port 8001
```

**Problem: `ImportError` on startup**

Missing or incompatible dependencies.

```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Check for missing packages
pip check
```

### API Request Failures

**Problem: `401 Unauthorized` responses**

Invalid or missing authentication token.

```bash
# Ensure Authorization header is set correctly
curl -H "Authorization: Bearer <your_token>" \
     http://localhost:8000/api/v1/generate
```

For authentication details, see [API Documentation]({% link docs/api-documentation.md %}#overview).

**Problem: `429 Too Many Requests`**

Rate limit exceeded.

```bash
# Check Retry-After header in response
# Wait the specified time before retrying

# Rate limits:
# - Per conversation: 1 request per 3 seconds
# - Per user: 20 requests per minute
```

For rate limiting details, see [API Documentation - Rate Limiting]({% link docs/api-documentation.md %}#rate-limiting).

**Problem: `408 Request Timeout` or `503 Service Unavailable`**

AI service timeout or unavailability.

The system implements retry logic with exponential backoff. If issues persist:

1. Check AI provider API key is valid
2. Verify network connectivity to AI provider
3. Review AI service logs for specific errors

```bash
# Check backend logs
tail -f logs/app.log
```

---

## Frontend Build/Runtime Errors

### Build Failures

**Problem: TypeScript compilation errors**

Type mismatches or missing type definitions.

```bash
# Run type checking
npx tsc --noEmit

# Check for missing types
npm install @types/<package_name> --save-dev
```

**Problem: `vite build` fails with memory error**

Node.js running out of memory during build.

```bash
# Increase Node.js memory limit
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build
```

### Runtime Errors

**Problem: Blank page or JavaScript errors in browser**

Check browser console for specific errors.

```bash
# Clear browser cache
# Open DevTools (F12) > Application > Clear storage

# Verify environment variables
cat .env | grep VITE_

# Rebuild the frontend
npm run build
npm run dev
```

**Problem: `CORS` errors in browser console**

Backend not configured to allow frontend origin.

Ensure the backend is running and CORS is properly configured. The frontend should be accessing the correct API URL specified in `VITE_API_URL`.

**Problem: `Failed to fetch` or network errors**

Backend is not running or incorrect API URL.

```bash
# Verify backend is running
curl http://localhost:8000/health

# Check frontend .env
cat .env | grep VITE_API_URL
# Should be: VITE_API_URL=http://localhost:8000
```

---

## Authentication Problems

### Login Issues

**Problem: Unable to log in or create account**

Neon Auth configuration issues.

1. Verify `NEON_AUTH_SECRET` in backend `.env`
2. Verify `VITE_AUTH_DOMAIN` in frontend `.env`
3. Check network connectivity to Neon Auth service

**Problem: Token expired or invalid**

JWT token has expired and needs refresh.

The frontend should automatically handle token refresh. If issues persist:

1. Clear browser local storage
2. Log out and log back in
3. Check that token refresh endpoint is accessible

### Session Issues

**Problem: Session data not persisting**

Database connection or transaction issues.

```bash
# Verify database connectivity
psql $DATABASE_URL -c "SELECT 1"

# Check for uncommitted transactions in backend logs
tail -f logs/app.log | grep -i "transaction"
```

---

## Common Error Messages

### Backend Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `ValidationError` | Invalid request data | Check request body matches API schema |
| `ForbiddenError` | Accessing another user's data | Verify user has permission for the resource |
| `NotFoundError` | Resource does not exist | Verify the conversation/user ID is correct |
| `ConflictError` | Duplicate resource | Resource already exists with that identifier |

For the complete error format, see [API Documentation - Error Handling]({% link docs/api-documentation.md %}#error-handling).

### Frontend Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `TypeError: Cannot read property of undefined` | Missing data or null reference | Add null checks and loading states |
| `Hydration mismatch` | Server/client HTML differs | Ensure consistent rendering between server and client |
| `ChunkLoadError` | Failed to load code chunk | Clear cache and reload, check network |

---

## Test Failures

### Frontend Tests

**Problem: Tests fail with `ReferenceError: document is not defined`**

Running browser tests in Node.js environment without DOM.

```bash
# Ensure test setup includes jsdom
# In vitest.config.ts:
# environment: 'jsdom'
```

**Problem: Tests timeout waiting for elements**

Async operations not completing.

```typescript
// Use proper async testing patterns
import { waitFor } from '@testing-library/react';

await waitFor(() => {
  expect(screen.getByText('Expected Text')).toBeInTheDocument();
});
```

### Backend Tests

**Problem: Tests fail with database errors**

Test database not configured or not isolated.

```bash
# Ensure test database exists
# Set TEST_DATABASE_URL in test configuration

# Use transactions for test isolation
@pytest.fixture
async def db_session():
    async with engine.begin() as conn:
        yield conn
        await conn.rollback()
```

For testing best practices, see [Testing Strategy]({% link docs/testing-strategy.md %}).

---

## Getting Help

If you cannot resolve an issue using this guide:

1. **Search existing issues** on the project repositories
2. **Check the logs** for detailed error messages
3. **Create a new issue** with:
   - Steps to reproduce
   - Expected vs actual behavior
   - Environment details (OS, versions)
   - Relevant log output

---

## Related Documentation

| Topic | Document |
|-------|----------|
| Initial setup | [Getting Started]({% link docs/getting-started.md %}) |
| Development environment | [Development Workflows]({% link docs/development-workflows.md %}) |
| API reference | [API Documentation]({% link docs/api-documentation.md %}) |
| Testing guidance | [Testing Strategy]({% link docs/testing-strategy.md %}) |
