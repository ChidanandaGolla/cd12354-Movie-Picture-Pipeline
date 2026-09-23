# ✅ CI/CD Pipeline Verification Checklist

## 📋 Rubric Compliance & Verification

### 1. Frontend Continuous Integration (`.github/workflows/frontend-ci.yml`)
- [x] Named "Frontend Continuous Integration"
- [x] Triggers on `pull_request` against `main` for `starter/frontend/**`
- [x] Supports manual trigger `workflow_dispatch`
- [x] Runs `lint` job with ESLint (`npm run lint`)
- [x] Runs `test` job with Jest (`CI=true npm test`)
- [x] `lint` and `test` jobs run in **parallel**
- [x] `build` job runs only when `lint` and `test` succeed (`needs: [lint, test]`)
- [x] `build` job builds Docker image using `starter/frontend/Dockerfile`

### 2. Frontend Continuous Deployment (`.github/workflows/frontend-cd.yml`)
- [x] Named "Frontend Continuous Deployment"
- [x] Triggers on `push` against `main` for `starter/frontend/**`
- [x] Supports manual trigger `workflow_dispatch`
- [x] Runs `lint` and `test` jobs before deployment
- [x] `build-and-push` job builds Docker image with `REACT_APP_MOVIE_API_URL`
- [x] Docker image is tagged with Git SHA (`${{ github.sha }}`)
- [x] Pushes image to Amazon ECR repository `frontend`
- [x] Uses GitHub Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `EKS_CLUSTER_NAME`)
- [x] `deploy` job runs `aws eks update-kubeconfig`
- [x] Sets image tag with `kustomize edit set image`
- [x] Deploys manifests to Kubernetes with `kustomize build | kubectl apply -f -`
- [x] Verifies rollout with `kubectl rollout status deployment/frontend`

### 3. Backend Continuous Integration (`.github/workflows/backend-ci.yml`)
- [x] Named "Backend Continuous Integration"
- [x] Triggers on `pull_request` against `main` for `starter/backend/**`
- [x] Supports manual trigger `workflow_dispatch`
- [x] Runs `lint` job with Flake8 (`flake8 .`)
- [x] Runs `test` job with Pytest (`pytest`)
- [x] `lint` and `test` jobs run in **parallel**
- [x] `build` job runs only when `lint` and `test` succeed (`needs: [lint, test]`)
- [x] `build` job builds Docker image using `starter/backend/Dockerfile`

### 4. Backend Continuous Deployment (`.github/workflows/backend-cd.yml`)
- [x] Named "Backend Continuous Deployment"
- [x] Triggers on `push` against `main` for `starter/backend/**`
- [x] Supports manual trigger `workflow_dispatch`
- [x] Runs `lint` and `test` jobs before deployment
- [x] `build-and-push` job builds Docker image with `starter/backend/Dockerfile`
- [x] Docker image is tagged with Git SHA (`${{ github.sha }}`)
- [x] Pushes image to Amazon ECR repository `backend`
- [x] Uses GitHub Secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `EKS_CLUSTER_NAME`)
- [x] `deploy` job runs `aws eks update-kubeconfig`
- [x] Sets image tag with `kustomize edit set image`
- [x] Deploys manifests to Kubernetes with `kustomize build | kubectl apply -f -`
- [x] Verifies rollout with `kubectl rollout status deployment/backend`

### 5. Infrastructure & Configuration
- [x] Terraform creates VPC, subnets, route tables, IGW
- [x] Terraform creates ECR repositories `frontend` and `backend`
- [x] Terraform creates EKS cluster `cluster` and managed node group
- [x] Terraform creates IAM user `github-action-user` with ECR/EKS permissions
- [x] CodeBuild repository location updated in `setup/terraform/main.tf`
- [x] `setup/init.sh` default IAM user set to `github-action-user`
- [x] Zero hardcoded AWS credentials in code or repository