---
layout: default
title: Implementation Guidance
nav_order: 6
parent: Developer Guide
---

# Implementation Guidance

This document provides high-level direction for implementing features in the Stakeholder Agent System. It serves as a map to help you navigate the codebase and understand where different types of changes belong.

---

## System Overview

For a complete understanding of the system design, including architecture diagrams and quality attributes, see [Architecture]({% link docs/architecture.md %}). For technology details, see [Technology Stack]({% link docs/technology-stack.md %}).

---

## Where to Make Changes

### Frontend Changes

The frontend handles user interface, client-side state, and API communication.

**When to modify the frontend:**
- Adding or modifying UI components
- Implementing client-side validation
- Managing local state or caching
- Handling authentication flows

**Key patterns:**
- Use functional components with hooks
- Follow the established component structure
- Implement proper error boundaries

See [Development Workflows - Frontend]({% link docs/development-workflows.md %}#frontend-typescriptreact) for coding standards.

---

### Backend Changes

The backend handles AI orchestration, business logic, and data persistence.

**When to modify the backend:**
- Adding new API endpoints
- Implementing AI prompt logic
- Adding validation or guardrails
- Managing database operations

**Key patterns:**
- Use dependency injection for services
- Apply async/await for I/O operations
- Implement proper error handling with custom exceptions

See [Development Workflows - Backend]({% link docs/development-workflows.md %}#backend-pythonfastapi) for coding standards.

---

### API Changes

When adding or modifying API endpoints:

1. Define the endpoint in the appropriate controller
2. Create request/response models with Pydantic
3. Implement proper validation and error handling
4. Update the API documentation

See [API Documentation]({% link docs/api-documentation.md %}) for the current endpoint reference and error handling patterns.

---

## Common Implementation Scenarios

### Adding a New Feature

1. **Review the architecture** - Understand which layers are affected
   See: [Architecture]({% link docs/architecture.md %})

2. **Create a feature branch** - Follow the branching convention
   See: [Development Workflows - Git Workflow]({% link docs/development-workflows.md %}#git-workflow)

3. **Implement backend logic** - Start with the API and service layer

4. **Implement frontend UI** - Connect to the backend API

5. **Write tests** - Cover both frontend and backend
   See: [Testing Strategy]({% link docs/testing-strategy.md %})

6. **Submit a pull request** - Follow the PR process
   See: [Development Workflows - Pull Request Process]({% link docs/development-workflows.md %}#pull-request-process)

---

### Working with AI Integration

The AI provider is accessed through pydantic AI, which abstracts the underlying provider details:

- Content guardrails and filtering
- Provider-specific prompt formatting
- Token budget management

**Key considerations:**
- Use the established Pydantic AI agent interface, not direct provider calls
- Handle `llm-Response-error` and `context-load-exception` gracefully

See [Architecture - AI Provider Abstraction]({% link docs/architecture.md %}#ai-provider-abstraction-layer) for design details.

---

### Working with the Database

All session state persists to Neon Postgres before acknowledgment.

**Key patterns:**
- Use connection pooling for performance
- Implement proper transaction management
- Follow the established data access patterns

**Persisted entities:**
- User accounts and profiles
- Session metadata and configuration
- Complete chat history
- Captured requirements and artifacts

See [Testing Strategy - Data Access Layer Tests]({% link docs/testing-strategy.md %}#data-access-layer-tests) for testing guidance.

---

### Authentication and Authorization

Authentication is handled by Neon Auth.

**Key patterns:**
- All API requests require a valid JWT token
- Implement tenant-level data isolation

See [API Documentation]({% link docs/api-documentation.md %}#overview) for authentication details.

---

## Quality Requirements

When implementing features, ensure your code meets these quality attributes:

| Attribute       | Target                              | Reference                                                                                  |
|-----------------|-------------------------------------|--------------------------------------------------------------------------------------------|
| Response time   | (unacheived) < 5 sec response time  | [Architecture - Performance]({% link docs/architecture.md %}#performance)                  |
| Test coverage   | 75%+ for services                   | [Testing Strategy]({% link docs/testing-strategy.md %}#test-coverage-requirements)         |
| Error handling  | Consistent error format             | [API Documentation - Error Handling]({% link docs/api-documentation.md %}#error-handling)  |

---

## Related Documentation

| Topic | Document |
|-------|----------|
| System design and quality attributes | [Architecture]({% link docs/architecture.md %}) |
| Technology decisions | [Technology Stack]({% link docs/technology-stack.md %}) |
| Coding standards and workflows | [Development Workflows]({% link docs/development-workflows.md %}) |
| Testing practices | [Testing Strategy]({% link docs/testing-strategy.md %}) |
| API reference | [API Documentation]({% link docs/api-documentation.md %}) |
| Common issues | [Troubleshooting]({% link docs/troubleshooting.md %}) |
| Contribution process | [Contributing]({% link docs/contributing.md %}) |
