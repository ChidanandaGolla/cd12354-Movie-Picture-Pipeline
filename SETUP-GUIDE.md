# Quick Setup Guide - CI/CD Pipeline

## Prerequisites

✅ GitHub repository with this code
✅ AWS Account with permissions (via Udacity Cloud Gateway or your AWS account) to provision:
   - VPC, Subnets, Routing, and Security Groups
   - ECR (Elastic Container Registry) for `frontend` and `backend`
   - EKS (Elastic Kubernetes Service) Cluster and Node Group
   - IAM User `github-action-user`

---

## Step 1: Provision AWS Infrastructure with Terraform

The infrastructure is pre-configured with Terraform in `setup/terraform`:

```bash
cd setup/terraform
terraform init
terraform apply
```

Type `yes` when prompted.

Take note of the Terraform outputs:
- `cluster_name` (default: `cluster`)
- `frontend_ecr` (e.g., `<account_id>.dkr.ecr.us-east-1.amazonaws.com/frontend`)
- `backend_ecr` (e.g., `<account_id>.dkr.ecr.us-east-1.amazonaws.com/backend`)
- `github_action_user_arn`

---

## Step 2: Generate Access Keys for `github-action-user`

1. Open the AWS Console and navigate to **IAM** → **Users**.
2. Select the `github-action-user` created by Terraform.
3. Go to the **Security credentials** tab.
4. Under **Access keys**, click **Create access key**.
5. Select **Application running outside AWS** and click **Next** → **Create access key**.
6. Copy both the **Access Key ID** and **Secret Access Key**.

---

## Step 3: Configure Kubernetes Permissions with `init.sh`

Run the initialization script in the `setup` directory to add `github-action-user` to Kubernetes RBAC (`aws-auth` ConfigMap):

```bash
cd setup
./init.sh
```

---

## Step 4: Configure GitHub Repository Secrets

In your GitHub repository:
1. Navigate to **Settings** → **Secrets and variables** → **Actions**.
2. Click **New repository secret** and add the following:

| Secret Name | Value | Description |
|---|---|---|
| `AWS_ACCESS_KEY_ID` | `AKIA...` | Access Key ID for `github-action-user` |
| `AWS_SECRET_ACCESS_KEY` | `wJalr...` | Secret Access Key for `github-action-user` |
| `EKS_CLUSTER_NAME` | `cluster` | Name of your EKS cluster from Terraform output |

---

## Step 5: Workflow Overview

The repository contains 4 automated GitHub Actions workflows in `.github/workflows/`:

```
.github/workflows/
├── frontend-ci.yml   # PR trigger: Lint & Test (parallel) -> Docker Build
├── frontend-cd.yml   # Push trigger: Lint & Test -> Docker Build -> Push to ECR -> Deploy to EKS
├── backend-ci.yml    # PR trigger: Lint & Test (parallel) -> Docker Build
└── backend-cd.yml    # Push trigger: Lint & Test -> Docker Build -> Push to ECR -> Deploy to EKS
```

---

## Step 6: Testing the Pipelines

### 1. Test CI Pipeline (Pull Request)
```bash
# Create and switch to a feature branch
git checkout -b test-frontend-ci

# Make a minor change in frontend
echo "// test ci" >> starter/frontend/src/index.tsx

# Commit and push
git add .
git commit -m "Test frontend CI pipeline"
git push origin test-frontend-ci
```
- Open a Pull Request against `main`.
- In the **Actions** tab, verify that `Frontend Continuous Integration` runs `lint` and `test` in parallel, followed by `build`.

### 2. Demonstrate CI Failure Handling
- Modify a test assertion in `starter/frontend` or set `FAIL_TEST=true` in `starter/backend`.
- Push to a PR branch.
- Verify that the workflow halts and marks the PR check as failed.

### 3. Test CD Pipeline (Push to Main)
- Merge a passing PR to `main`.
- In the **Actions** tab, watch `Frontend Continuous Deployment` or `Backend Continuous Deployment` run:
  1. `lint` & `test` (parallel)
  2. `build-and-push` (builds Docker image, tags with Git SHA, pushes to ECR)
  3. `deploy` (updates kubeconfig, updates manifest via kustomize, deploys to EKS, verifies rollout)

---

## Step 7: Verify Kubernetes Deployment

```bash
# Verify pods and services
kubectl get pods -n default
kubectl get svc -n default

# Verify rollout status
kubectl rollout status deployment/frontend -n default
kubectl rollout status deployment/backend -n default
```

---

## Step 8: Clean Up Resources to Avoid AWS Charges

> [!WARNING]
> Running EKS clusters and LoadBalancers incurs ongoing AWS costs. Tear down infrastructure immediately after verifying deployments:

```bash
cd setup/terraform
terraform destroy
```
Type `yes` when prompted.