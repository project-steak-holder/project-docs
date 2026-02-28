---
layout: default
title: Technology Stack
nav_order: 3
parent: Developer Guide
---

# Technology Stack

This document outlines the technologies used in the Stakeholder Agent System and the rationale behind each selection.

## Overview

| Layer          | Technology                          | Purpose                                  |
|----------------|-------------------------------------|------------------------------------------|
| Front-End      | TypeScript, TanStack Start, Node.js | Browser-based chat interface             |
| AI Service     | Python, Pydantic AI, FastAPI        | Prompt orchestration, AI guardrails      |
| Data Store     | Neon Postgres                       | Persistent storage                       |
| Communication  | REST APIs                           | Service integration                      |
| Authentication | Neon Auth                           | Identity and session management          |
| Hosting        | Google Cloud Platform               | Cloud infrastructure                     |

## Frontend

### TypeScript

TypeScript provides static typing for the browser-based application, improving code reliability and developer experience through compile-time error detection and enhanced IDE support.

### TanStack Start

TanStack Start serves as the foundation for the responsive chat interface. This framework was selected for:

- Modern React-based architecture with server-side rendering capabilities
- Type-safe routing and data fetching
- Optimized client-side state management for real-time chat interactions

### Node.js

Node.js is used for the frontend build and development environment, enabling:

- Efficient package management with npm
- Fast development server with hot module replacement
- Compatibility with modern TypeScript tooling and libraries
- Seamless integration with TanStack Start and other frontend dependencies

## Backend / AI Service

### Python

Python serves as the primary language for the AI Service tier. The choice supports:

- Extensive AI/ML ecosystem compatibility
- Rapid development for prompt engineering workflows
- Strong async I/O support for non-blocking AI provider calls

### Pydantic AI

Pydantic AI provides the framework for AI orchestration, offering:

- Structured prompt management and validation
- Type-safe data models for AI interactions
- Built-in support for context assembly and guardrails
- Token budget management and rate limiting capabilities

### FastAPI

FastAPI is the web framework for the backend service, chosen for:
- High performance with async support
- Automatic OpenAPI documentation generation
- Easy integration with Pydantic for request validation
- Developer-friendly features that enhance productivity and maintainability
- Seamless injection of Neon Auth for authentication and session management

## Data Storage

### Neon Postgres

Neon is a serverless Postgres platform that provides durable persistence for all application state. This choice supports:

- ACID compliance for session data integrity
- Structured storage for user accounts, session metadata, and chat history
- Connection pooling for optimized performance
- Reliable state storage ensuring session continuity
- Serverless scaling that adapts to demand

For details on persisted entities and data flow, see [Architecture]({% link docs/architecture.md %}#data-store).

## Infrastructure

### Google Cloud Platform

GCP provides the hosting infrastructure with:

- Scalable compute resources for the stateless backend design
- Network security and TLS encryption
- Health monitoring and logging capabilities

### REST API Communication

The system uses REST APIs with JSON payloads for service integration:

- Clear contract definition between front-end and AI service
- Broad tooling support for testing and debugging
- Familiar patterns that support team productivity

### Neon Auth

Neon Auth provides authentication and session management as part of the Neon platform, which:

- Reduces security implementation complexity
- Integrates seamlessly with Neon Postgres for unified data and identity management
- Simplifies compliance with security standards
- Handles token management and session validation

## Architectural Alignment

The technology selections support the system's primary quality attributes:

| Quality Attribute | Technology Support |
|-------------------|-------------------|
| **Availability** | Stateless Python backend enables horizontal scaling; Neon Postgres provides durability with serverless scaling |
| **Performance** | Async I/O in Python; connection pooling; TanStack Start client optimization |
| **Modifiability** | Pydantic AI abstraction layer allows provider substitution; data-driven configuration |
| **Security** | Neon Auth for identity management; TLS throughout; Pydantic validation for input sanitization |

---

## Next Steps

| Goal | Documentation |
|------|---------------|
| Understand system architecture | [Architecture]({% link docs/architecture.md %}) |
| Set up local development | [Getting Started]({% link docs/getting-started.md %}) |
| Learn coding standards | [Development Workflows]({% link docs/development-workflows.md %}) |
