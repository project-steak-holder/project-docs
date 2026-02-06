---
layout: default
title: Contributing
nav_order: 11
---

# Contributing

Thank you for your interest in contributing to the Stakeholder Agent System. This guide explains how to contribute effectively to the project.

---

## Getting Started

Before contributing, ensure you have:

1. Set up your local development environment - see [Getting Started]({% link docs/getting-started.md %})
2. Familiarized yourself with the codebase - see [Architecture]({% link docs/architecture.md %})
3. Reviewed the coding standards - see [Development Workflows]({% link docs/development-workflows.md %})

---

## Types of Contributions

We welcome several types of contributions:

| Contribution Type | Description |
|-------------------|-------------|
| Bug fixes | Resolving reported issues |
| Features | New functionality aligned with project goals |
| Documentation | Improvements to guides, API docs, or code comments |
| Tests | Additional test coverage or test improvements |
| Performance | Optimizations that improve system responsiveness |

---

## Branching Strategy

We follow a feature branch workflow. For complete branching details, see [Development Workflows - Git Workflow]({% link docs/development-workflows.md %}#git-workflow).

### Branch Naming Convention

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/<name>` | `feature/user-authentication` |
| Bug Fix | `bugfix/<issue>` | `bugfix/login-redirect` |
| Hotfix | `hotfix/<critical>` | `hotfix/security-patch` |
| Documentation | `docs/<topic>` | `docs/api-examples` |

### Creating a Branch

```bash
# Ensure you're on the latest main
git checkout main
git pull origin main

# Create your feature branch
git checkout -b feature/your-feature-name
```

---

## Development Process

### 1. Plan Your Changes

Before writing code:

- Check for existing issues or discussions related to your change
- For significant features, open an issue first to discuss the approach
- Review the [Architecture]({% link docs/architecture.md %}) to understand which components are affected

### 2. Implement Your Changes

Follow the coding standards for the relevant layer:

- **Frontend**: See [Development Workflows - Frontend]({% link docs/development-workflows.md %}#frontend-typescriptreact)
- **Backend**: See [Development Workflows - Backend]({% link docs/development-workflows.md %}#backend-pythonfastapi)
- **API Changes**: See [Implementation Guidance - API Changes]({% link docs/implementation-guidance.md %}#api-changes)

### 3. Write Tests

All contributions must include appropriate tests.

**Test Coverage Requirements:**

| Component | Minimum Coverage |
|-----------|------------------|
| Frontend Business Logic | 80% |
| Backend Services | 80% |
| API Endpoints | 100% |
| Data Access Layer | 90% |

For testing guidance, see [Testing Strategy]({% link docs/testing-strategy.md %}).

**Running Tests:**

```bash
# Frontend
cd ai-frontend
npm run test
npm run coverage

# Backend
cd ai-backend
pytest --cov=app
```

### 4. Commit Your Changes

#### Commit Message Format

We follow the Conventional Commits specification:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**

| Type | Description |
|------|-------------|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation changes |
| `style` | Formatting, missing semicolons, etc. (no code change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Build process or auxiliary tool changes |

**Examples:**

```bash
# Feature commit
git commit -m "feat(auth): add password reset functionality"

# Bug fix commit
git commit -m "fix(api): handle null user_id in generate endpoint"

# Documentation commit
git commit -m "docs(api): add rate limiting examples"

# Test commit
git commit -m "test(auth): add unit tests for token validation"
```

**Commit Message Guidelines:**

- Use the imperative mood ("add" not "added")
- Keep the description under 72 characters
- Reference issues when applicable: `fix(login): resolve redirect loop (closes #123)`

---

## Pull Request Process

### 1. Prepare Your PR

Before submitting:

- Ensure all tests pass locally
- Run linting and formatting checks
- Rebase on the latest main branch

```bash
# Update your branch with latest main
git fetch origin
git rebase origin/main

# Run all checks
# Frontend
npm run lint
npm run test

# Backend
ruff check .
pytest
```

### 2. Submit Your PR

```bash
# Push your branch
git push origin feature/your-feature-name
```

Create a pull request on GitHub with:

- **Title**: Clear, descriptive title following commit message format
- **Description**: Explain what the PR does and why
- **Testing**: Describe how to test the changes
- **Related Issues**: Link any related issues using `Closes #123`

**PR Description Template:**

```markdown
## Summary
Brief description of the changes.

## Changes
- List of specific changes made
- Another change

## Testing
Steps to test the changes:
1. Step one
2. Step two

## Related Issues
Closes #123
```

### 3. Address Review Feedback

- Respond to all reviewer comments
- Make requested changes in new commits
- Request re-review when ready

```bash
# Make changes and commit
git add .
git commit -m "fix: address review feedback"
git push origin feature/your-feature-name
```

---

## Code Review Expectations

### For Contributors

When submitting a PR:

- Keep PRs focused and reasonably sized
- Ensure CI passes before requesting review
- Be responsive to feedback
- Be open to suggestions and alternative approaches

### For Reviewers

When reviewing a PR:

- Review within 2 business days when possible
- Provide constructive, specific feedback
- Distinguish between required changes and suggestions
- Approve when the code meets standards

**Review Checklist:**

| Area | Criteria |
|------|----------|
| Functionality | Does the code work as intended? |
| Tests | Are there adequate tests? Do they pass? |
| Code Quality | Does it follow coding standards? |
| Documentation | Are changes documented appropriately? |
| Security | Are there any security concerns? |
| Performance | Are there any performance implications? |

---

## Merge Requirements

PRs can be merged when:

- All CI checks pass
- At least 1 approving review is received
- All review feedback is addressed
- Branch is up to date with main

After merge:

- The feature branch will be deleted
- Changes will be deployed to the development environment

---

## Documentation Contributions

### Types of Documentation

| Type | Location | Guidelines |
|------|----------|------------|
| API Documentation | `docs/api-documentation.md` | Include request/response examples |
| Architecture | `docs/architecture.md` | Update diagrams when adding components |
| Guides | `docs/` | Follow existing structure and formatting |
| Code Comments | Source files | Document public APIs and complex logic |

### Documentation Standards

- Use clear, concise language
- Include code examples where helpful
- Keep formatting consistent with existing docs
- Update the table of contents if adding sections

---

## Issue Reporting

### Bug Reports

When reporting a bug, include:

1. **Summary**: Brief description of the issue
2. **Steps to Reproduce**: Detailed steps to reproduce the bug
3. **Expected Behavior**: What you expected to happen
4. **Actual Behavior**: What actually happened
5. **Environment**: OS, browser, versions
6. **Screenshots/Logs**: If applicable

### Feature Requests

When requesting a feature:

1. **Problem**: Describe the problem you're trying to solve
2. **Proposed Solution**: Your idea for how to solve it
3. **Alternatives**: Other approaches you've considered
4. **Additional Context**: Any other relevant information

---

## Questions and Support

- For questions about contributing, open a discussion on GitHub
- For technical issues, see [Troubleshooting]({% link docs/troubleshooting.md %})
- For urgent issues, contact the maintainers directly

---

## Related Documentation

| Topic | Document |
|-------|----------|
| Environment setup | [Getting Started]({% link docs/getting-started.md %}) |
| Coding standards | [Development Workflows]({% link docs/development-workflows.md %}) |
| Testing requirements | [Testing Strategy]({% link docs/testing-strategy.md %}) |
| API reference | [API Documentation]({% link docs/api-documentation.md %}) |
| Common issues | [Troubleshooting]({% link docs/troubleshooting.md %}) |
