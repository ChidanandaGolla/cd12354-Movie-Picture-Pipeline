# CI/CD Pipeline Documentation

## Overview

This project implements a fully automated CI/CD pipeline using GitHub Actions for both the frontend (React/Node.js) and backend (Flask/Python) applications of the Movie Picture application.

---

## Workflow Architecture

### 1. Frontend Continuous Integration (`.github/workflows/frontend-ci.yml`)
- **Trigger**: Pull Requests against `main` when files in `starter/frontend/**` change, or manual trigger (`workflow_dispatch`).
- **Jobs**:
  1. `lint` (ESLint): Validates code style and rules. Runs in parallel with `test`.
  2. `test` (Jest): Runs unit tests with `CI=true npm test`. Runs in parallel with `lint`.
  3. `build` (Docker): Triggered only when both `lint` and `test` succeed (`needs: [lint, test]`). Builds Docker image using `starter/frontend/Dockerfile`.

### 2. Frontend Continuous Deployment (`.github/workflows/frontend-cd.yml`)
- **Trigger**: Pushes to `main` when files in `starter/frontend/**` change, or manual trigger (`workflow_dispatch`).
- **Jobs**:
  1. `lint` & `test`: Parallel validation jobs.
  2. `build-and-push`: Runs on `needs: [lint, test]`. Logs into AWS ECR, builds Docker image with `--build-arg REACT_APP_MOVIE_API_URL=...`, tags with Git SHA (`${{ github.sha }}`), and pushes to ECR repository `frontend`.
  3. `deploy`: Runs on `needs: [build-and-push]`. Configures `kubectl` with `aws eks update-kubeconfig`, updates container image in `starter/frontend/k8s` with `kustomize edit set image frontend=<ECR_IMAGE_URI>:<GIT_SHA>`, applies manifests with `kustomize build | kubectl apply -f -`, and checks rollout status.

### 3. Backend Continuous Integration (`.github/workflows/backend-ci.yml`)
- **Trigger**: Pull Requests against `main` when files in `starter/backend/**` change, or manual trigger (`workflow_dispatch`).
- **Jobs**:
  1. `lint` (Flake8): Checks PEP8 compliance. Runs in parallel with `test`.
  2. `test` (Pytest): Runs Python unit tests. Runs in parallel with `lint`.
  3. `build` (Docker): Triggered only when both `lint` and `test` succeed (`needs: [lint, test]`). Builds Docker image using `starter/backend/Dockerfile`.

### 4. Backend Continuous Deployment (`.github/workflows/backend-cd.yml`)
- **Trigger**: Pushes to `main` when files in `starter/backend/**` change, or manual trigger (`workflow_dispatch`).
- **Jobs**:
  1. `lint` & `test`: Parallel validation jobs.
  2. `build-and-push`: Runs on `needs: [lint, test]`. Logs into AWS ECR, builds Docker image, tags with Git SHA (`${{ github.sha }}`), and pushes to ECR repository `backend`.
  3. `deploy`: Runs on `needs: [build-and-push]`. Configures `kubectl` with `aws eks update-kubeconfig`, updates container image in `starter/backend/k8s` with `kustomize edit set image backend=<ECR_IMAGE_URI>:<GIT_SHA>`, applies manifests with `kustomize build | kubectl apply -f -`, and checks rollout status.

---

## Required GitHub Secrets

Configure these in your GitHub repository (**Settings** → **Secrets and variables** → **Actions**):

| Secret Name | Purpose | Example |
|---|---|---|
| `AWS_ACCESS_KEY_ID` | IAM User Access Key | `AKIA...` |
| `AWS_SECRET_ACCESS_KEY` | IAM User Secret Key | `wJalrXUtn...` |
| `EKS_CLUSTER_NAME` | Name of EKS Cluster | `cluster` |

---

## Local Testing Commands

### Frontend
```bash
cd starter/frontend

# Install dependencies
npm ci

# Run linting
npm run lint

# Run tests
CI=true npm test

# Test failure simulation
FAIL_TEST=true CI=true npm test
```

### Backend
```bash
cd starter/backend

# Install dependencies
pipenv install --dev

# Run linting
pipenv run lint

# Run tests
pipenv run test

# Test failure simulation
FAIL_TEST=true pipenv run test
```

---

## Kubernetes Deployment Flow

Manifests located in `starter/frontend/k8s` and `starter/backend/k8s` use `kustomize` for declarative image version injection:

```bash
# Update image tag dynamically
kustomize edit set image frontend=<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/frontend:<GIT_SHA>

# Apply to cluster
kustomize build | kubectl apply -f -

# Verify rollout
kubectl rollout status deployment/frontend -n default --timeout=5m
```