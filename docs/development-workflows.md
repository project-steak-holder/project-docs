# Development Workflows

## Development Process

### Setting Up Your Development Environment

1. **Fork the Repository**
   ```bash
   # Fork via GitHub UI, then clone your fork
   git clone https://github.com/YOUR_USERNAME/project-name.git
   cd project-name
   ```

2. **Configure Upstream**
   ```bash
   git remote add upstream https://github.com/original-owner/project-name.git
   ```

3. **Install Development Dependencies**
   ```bash
   [installation command]
   ```

## Branching Strategy

### Main Branches

- **`main`** - Production-ready code
- **`develop`** - Integration branch for features

### Feature Branches

Create feature branches from `develop`:

```bash
git checkout develop
git pull upstream develop
git checkout -b feature/your-feature-name
```

### Branch Naming Convention

- **Features:** `feature/description`
- **Bug Fixes:** `fix/description`
- **Hotfixes:** `hotfix/description`
- **Documentation:** `docs/description`
- **Refactoring:** `refactor/description`

## Development Workflow

### 1. Create a Feature Branch

```bash
git checkout -b feature/new-feature
```

### 2. Make Changes

- Write code following [coding standards](./implementation-guidance.md)
- Write tests for new functionality
- Update documentation as needed

### 3. Commit Changes

Follow conventional commit format:

```bash
git commit -m "type(scope): description

Optional body with more details.

Fixes #123"
```

**Commit Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### 4. Keep Branch Updated

```bash
git fetch upstream
git rebase upstream/develop
```

### 5. Push Changes

```bash
git push origin feature/new-feature
```

### 6. Create Pull Request

- Go to GitHub and create a PR from your feature branch to `develop`
- Fill out the PR template
- Link related issues
- Request reviews

## Code Review Process

1. **Submit PR** - Create a pull request with clear description
2. **Automated Checks** - CI/CD runs automated tests
3. **Peer Review** - Team members review code
4. **Address Feedback** - Make requested changes
5. **Approval** - Get required approvals
6. **Merge** - Maintainer merges the PR

## Local Development

### Running Locally

```bash
# Start development server
[dev server command]
```

### Hot Reload

The development server supports hot reload for faster iteration.

### Environment Variables

Create a `.env` file for local development:

```env
# Example environment variables
DATABASE_URL=postgresql://localhost/dbname
API_KEY=your-api-key
DEBUG=true
```

## Testing During Development

### Run Unit Tests

```bash
[unit test command]
```

### Run Integration Tests

```bash
[integration test command]
```

### Run with Watch Mode

```bash
[test watch command]
```

## Debugging

### Local Debugging

[Instructions for debugging locally]

### Remote Debugging

[Instructions for remote debugging if applicable]

## Code Quality Tools

### Linting

```bash
# Run linter
[lint command]

# Fix auto-fixable issues
[lint fix command]
```

### Formatting

```bash
# Check formatting
[format check command]

# Auto-format code
[format command]
```

### Type Checking

```bash
# Run type checker
[type check command]
```

## Release Workflow

1. **Create Release Branch**
   ```bash
   git checkout -b release/v1.0.0 develop
   ```

2. **Update Version Numbers**
   - Update version in package files
   - Update CHANGELOG

3. **Create Release PR**
   - PR from release branch to `main`
   - Get approvals

4. **Tag Release**
   ```bash
   git tag -a v1.0.0 -m "Release version 1.0.0"
   git push origin v1.0.0
   ```

5. **Merge Back to Develop**
   ```bash
   git checkout develop
   git merge main
   ```

---

**Next:** Review the [Testing Strategy](./testing-strategy.md).
