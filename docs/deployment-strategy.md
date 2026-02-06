---
layout: default
title: Deployment Strategy
nav_order: 9
---

# Deployment Strategy

This document outlines the comprehensive deployment strategy for the Stakeholder Agent System using Jenkins, Docker, and AWS infrastructure.

## Overview

The deployment strategy supports the system's multi-tier architecture:

- **Frontend**: TypeScript/TanStack Start application
- **Backend**: Python/Pydantic AI service
- **Database**: Neon Postgres (managed service)

| Component | Containerization | Orchestration | Registry |
| --------- | ---------------- | ------------- | -------- |
| Frontend  | Docker           | AWS ECS       | AWS ECR  |
| Backend   | Docker           | AWS ECS       | AWS ECR  |
| Database  | Managed          | Neon          | N/A      |

---

## Docker Configuration

### Backend Dockerfile

```dockerfile
# ai-backend/Dockerfile
FROM python:3.11-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Production stage
FROM python:3.11-slim AS production

WORKDIR /app

# Create non-root user
RUN useradd --create-home --shell /bin/bash appuser

# Copy installed packages from builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy application code
COPY --chown=appuser:appuser . .

# Set environment
ENV PATH=/home/appuser/.local/bin:$PATH
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Frontend Dockerfile

```dockerfile
# ai-frontend/Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production=false

# Copy source and build
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Copy built assets and dependencies
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/package.json ./

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### Docker Compose for Local Development

```yaml
# docker-compose.yml
version: "3.8"

services:
  frontend:
    build:
      context: ./ai-frontend
      target: production
    ports:
      - "5173:3000"
    environment:
      - VITE_API_URL=http://backend:8000
      - VITE_AUTH_DOMAIN=${VITE_AUTH_DOMAIN}
    depends_on:
      backend:
        condition: service_healthy
    networks:
      - stakeholder-network

  backend:
    build:
      context: ./ai-backend
      target: production
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - NEON_AUTH_SECRET=${NEON_AUTH_SECRET}
      - AI_PROVIDER_API_KEY=${AI_PROVIDER_API_KEY}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - stakeholder-network

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=stakeholder
      - POSTGRES_PASSWORD=localdev
      - POSTGRES_DB=stakeholder_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - stakeholder-network

networks:
  stakeholder-network:
    driver: bridge

volumes:
  postgres_data:
```

---

## AWS Infrastructure

### Architecture Diagram

```
                                    +------------------+
                                    |   Route 53       |
                                    |   (DNS)          |
                                    +--------+---------+
                                             |
                                    +--------v---------+
                                    |   CloudFront     |
                                    |   (CDN)          |
                                    +--------+---------+
                                             |
                         +-------------------+-------------------+
                         |                                       |
                +--------v---------+                    +--------v---------+
                |   ALB (Public)   |                    |   S3 Bucket      |
                |   Port 443       |                    |   (Static Assets)|
                +--------+---------+                    +------------------+
                         |
         +---------------+---------------+
         |                               |
+--------v---------+            +--------v---------+
|   ECS Cluster    |            |   ECS Cluster    |
|   (Frontend)     |            |   (Backend)      |
|   Private Subnet |            |   Private Subnet |
+------------------+            +--------+---------+
                                         |
                         +---------------+---------------+
                         |                               |
                +--------v---------+            +--------v---------+
                |   Neon Postgres  |            |   ElastiCache    |
                |   (External)     |            |   (Redis)        |
                +------------------+            +------------------+
```

### VPC Design

```
VPC: 10.0.0.0/16
|
+-- Public Subnets (ALB, NAT Gateway)
|   +-- 10.0.1.0/24 (us-east-1a)
|   +-- 10.0.2.0/24 (us-east-1b)
|   +-- 10.0.3.0/24 (us-east-1c)
|
+-- Private Subnets (ECS Tasks)
|   +-- 10.0.11.0/24 (us-east-1a)
|   +-- 10.0.12.0/24 (us-east-1b)
|   +-- 10.0.13.0/24 (us-east-1c)
|
+-- Data Subnets (ElastiCache)
    +-- 10.0.21.0/24 (us-east-1a)
    +-- 10.0.22.0/24 (us-east-1b)
```

### AWS Services

| Service         | Purpose                 | Configuration         |
| --------------- | ----------------------- | --------------------- |
| ECS Fargate     | Container orchestration | Serverless compute    |
| ECR             | Container registry      | Image storage         |
| ALB             | Load balancing          | HTTPS termination     |
| CloudFront      | CDN                     | Static asset delivery |
| Route 53        | DNS                     | Domain management     |
| ElastiCache     | Session caching         | Redis cluster         |
| Secrets Manager | Secret storage          | API keys, credentials |
| CloudWatch      | Monitoring              | Logs, metrics, alarms |
| WAF             | Security                | Request filtering     |

### Environment Separation

| Environment | AWS Account     | Purpose         | Access     |
| ----------- | --------------- | --------------- | ---------- |
| Development | dev-account     | Feature testing | Developers |
| Staging     | staging-account | Pre-production  | QA Team    |
| Production  | prod-account    | Live system     | Restricted |

### Security Configuration

**Security Groups**

| Group       | Inbound                    | Outbound | Purpose      |
| ----------- | -------------------------- | -------- | ------------ |
| alb-sg      | 443 (0.0.0.0/0)            | All      | Public ALB   |
| frontend-sg | 3000 (alb-sg)              | All      | Frontend ECS |
| backend-sg  | 8000 (frontend-sg, alb-sg) | All      | Backend ECS  |
| cache-sg    | 6379 (backend-sg)          | None     | ElastiCache  |

**IAM Roles**

```
ECS Task Execution Role:
- ecr:GetAuthorizationToken
- ecr:BatchCheckLayerAvailability
- ecr:GetDownloadUrlForLayer
- ecr:BatchGetImage
- logs:CreateLogStream
- logs:PutLogEvents
- secretsmanager:GetSecretValue

ECS Task Role:
- s3:GetObject (assets bucket)
- secretsmanager:GetSecretValue
- cloudwatch:PutMetricData
```

---

## Jenkins Pipeline

### Jenkinsfile

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        FRONTEND_REPO = 'stakeholder-frontend'
        BACKEND_REPO = 'stakeholder-backend'
        IMAGE_TAG = "${env.GIT_COMMIT?.take(8) ?: 'latest'}"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        dir('ai-frontend') {
                            sh 'npm ci'
                            sh 'npm run build'
                        }
                    }
                }
                stage('Build Backend') {
                    steps {
                        dir('ai-backend') {
                            sh 'python -m venv venv'
                            sh '. venv/bin/activate && pip install -r requirements.txt'
                        }
                    }
                }
            }
        }

        stage('Test') {
            parallel {
                stage('Test Frontend') {
                    steps {
                        dir('ai-frontend') {
                            sh 'npm run test -- --coverage'
                        }
                    }
                    post {
                        always {
                            publishHTML(target: [
                                reportDir: 'ai-frontend/coverage',
                                reportFiles: 'index.html',
                                reportName: 'Frontend Coverage'
                            ])
                        }
                    }
                }
                stage('Test Backend') {
                    steps {
                        dir('ai-backend') {
                            sh '. venv/bin/activate && pytest --cov=app --cov-report=html'
                        }
                    }
                    post {
                        always {
                            publishHTML(target: [
                                reportDir: 'ai-backend/htmlcov',
                                reportFiles: 'index.html',
                                reportName: 'Backend Coverage'
                            ])
                        }
                    }
                }
            }
        }

        stage('Security Scan') {
            parallel {
                stage('SAST Scan') {
                    steps {
                        sh 'bandit -r ai-backend/app -f json -o bandit-report.json || true'
                    }
                }
                stage('Dependency Scan') {
                    steps {
                        sh 'npm audit --prefix ai-frontend --json > npm-audit.json || true'
                        sh '. ai-backend/venv/bin/activate && pip-audit --format json > pip-audit.json || true'
                    }
                }
            }
        }

        stage('Build Docker Images') {
            when {
                anyOf {
                    branch 'main'
                    branch 'staging'
                    branch pattern: "release/*", comparator: "GLOB"
                }
            }
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """

                    sh """
                        docker build -t ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG} \
                            --target production ai-frontend/
                        docker push ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG}
                    """

                    sh """
                        docker build -t ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG} \
                            --target production ai-backend/
                        docker push ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Development') {
            when {
                branch 'main'
            }
            steps {
                script {
                    deployToEnvironment('development', IMAGE_TAG)
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    deployToEnvironment('staging', IMAGE_TAG)
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch pattern: "release/*", comparator: "GLOB"
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                script {
                    deployToEnvironment('production', IMAGE_TAG)
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: 'good',
                message: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}

def deployToEnvironment(String environment, String imageTag) {
    sh """
        aws ecs update-service \
            --cluster stakeholder-${environment} \
            --service frontend-service \
            --force-new-deployment \
            --region ${AWS_REGION}

        aws ecs update-service \
            --cluster stakeholder-${environment} \
            --service backend-service \
            --force-new-deployment \
            --region ${AWS_REGION}
    """

    sh """
        aws ecs wait services-stable \
            --cluster stakeholder-${environment} \
            --services frontend-service backend-service \
            --region ${AWS_REGION}
    """
}
```

### Pipeline Stages Summary

| Stage          | Actions                       | Triggers                  |
| -------------- | ----------------------------- | ------------------------- |
| Build          | Install dependencies, compile | All branches              |
| Test           | Unit tests, integration tests | All branches              |
| Security Scan  | SAST, dependency audit        | All branches              |
| Docker Build   | Build and push images         | main, staging, release/\* |
| Deploy Dev     | Deploy to development         | main branch               |
| Deploy Staging | Deploy to staging             | staging branch            |
| Deploy Prod    | Deploy to production (manual) | release/\* branches       |

---

## Deployment Workflow

### Branch Strategy

```
main (development)
  |
  +-- feature/* (development work)
  |     |
  |     +-- PR --> main (auto-deploy to dev)
  |
  +-- staging (pre-production)
  |     |
  |     +-- PR from main --> staging (auto-deploy to staging)
  |
  +-- release/* (production releases)
        |
        +-- PR from staging --> release/v1.x (manual deploy to prod)
```

### Deployment Process

| Environment | Trigger            | Approval  | Strategy   |
| ----------- | ------------------ | --------- | ---------- |
| Development | Push to main       | Automatic | Rolling    |
| Staging     | Push to staging    | Automatic | Blue-Green |
| Production  | Push to release/\* | Manual    | Blue-Green |

### Blue-Green Deployment

```
                    +------------------+
                    |   ALB            |
                    |   (100% Blue)    |
                    +--------+---------+
                             |
         +-------------------+-------------------+
         |                                       |
+--------v---------+                    +--------v---------+
|   Blue (Active)  |                    |   Green (Idle)   |
|   v1.0.0         |                    |   v1.1.0         |
+------------------+                    +------------------+

Deployment Steps:
1. Deploy new version to Green
2. Run health checks on Green
3. Shift 10% traffic to Green (canary)
4. Monitor for 5 minutes
5. Shift 100% traffic to Green
6. Keep Blue as rollback target
```

### Rollback Procedure

**Automatic Rollback Triggers:**

- Health check failures (3 consecutive)
- Error rate > 5% for 2 minutes
- Response time p95 > 5 seconds

**Manual Rollback:**

```bash
# Rollback to previous task definition
aws ecs update-service \
    --cluster stakeholder-production \
    --service backend-service \
    --task-definition stakeholder-backend:PREVIOUS_REVISION \
    --force-new-deployment

# Or switch ALB target group (blue-green)
aws elbv2 modify-listener \
    --listener-arn $LISTENER_ARN \
    --default-actions Type=forward,TargetGroupArn=$BLUE_TARGET_GROUP_ARN
```

---

## Health Checks and Monitoring

### Health Check Endpoints

| Service  | Endpoint    | Interval | Threshold  |
| -------- | ----------- | -------- | ---------- |
| Frontend | GET /health | 30s      | 3 failures |
| Backend  | GET /health | 30s      | 3 failures |

### CloudWatch Alarms

| Alarm           | Metric             | Threshold | Action           |
| --------------- | ------------------ | --------- | ---------------- |
| HighErrorRate   | 5XX errors         | > 5%      | SNS notification |
| HighLatency     | p95 response time  | > 5s      | SNS notification |
| LowHealthyHosts | Healthy host count | < 2       | Auto-scaling     |
| HighCPU         | CPU utilization    | > 80%     | Auto-scaling     |

### Monitoring Dashboard

```
+------------------------------------------------------------------+
|                    Stakeholder System Dashboard                   |
+------------------------------------------------------------------+
| Request Rate        | Error Rate          | Response Time (p95)  |
| [===========] 150/s | [==] 0.5%           | [====] 250ms         |
+------------------------------------------------------------------+
| CPU Utilization     | Memory Utilization  | Active Connections   |
| [======] 45%        | [=====] 52%         | [=======] 89         |
+------------------------------------------------------------------+
| Healthy Hosts       | Deployments (24h)   | Incidents (24h)      |
| Frontend: 3/3       | 2                   | 0                    |
| Backend: 3/3        |                     |                      |
+------------------------------------------------------------------+
```

---

## Environment Variables

### Backend Service

| Variable            | Description              | Source          |
| ------------------- | ------------------------ | --------------- |
| DATABASE_URL        | Neon Postgres connection | Secrets Manager |
| NEON_AUTH_SECRET    | Authentication secret    | Secrets Manager |
| AI_PROVIDER_API_KEY | LLM provider API key     | Secrets Manager |
| ENVIRONMENT         | dev/staging/production   | Task definition |
| LOG_LEVEL           | Logging verbosity        | Task definition |

### Frontend Service

| Variable         | Description            | Source          |
| ---------------- | ---------------------- | --------------- |
| VITE_API_URL     | Backend API endpoint   | Task definition |
| VITE_AUTH_DOMAIN | Neon Auth domain       | Task definition |
| ENVIRONMENT      | dev/staging/production | Task definition |

---

## Related Documentation

| Topic                 | Document                                                          |
| --------------------- | ----------------------------------------------------------------- |
| System architecture   | [Architecture]({% link docs/architecture.md %})                   |
| Technology decisions  | [Technology Stack]({% link docs/technology-stack.md %})           |
| Development workflows | [Development Workflows]({% link docs/development-workflows.md %}) |
| Testing practices     | [Testing Strategy]({% link docs/testing-strategy.md %})           |
| API reference         | [API Documentation]({% link docs/api-documentation.md %})         |
