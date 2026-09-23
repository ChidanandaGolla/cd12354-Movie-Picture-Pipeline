# 📖 CI/CD Pipeline - Documentation Index

## 🚀 Overview

This repository provides an automated CI/CD pipeline for the Movie Picture application using GitHub Actions, AWS ECR, and AWS EKS.

## 📂 Documentation Directory

1. [VERIFICATION-CHECKLIST.md](file:///c:/Users/cnyad/OneDrive/Desktop/clone/cd12354-Movie-Picture-Pipeline/VERIFICATION-CHECKLIST.md)
   - Detailed rubric compliance checklist
   - Status of CI and CD requirements
2. [SETUP-GUIDE.md](file:///c:/Users/cnyad/OneDrive/Desktop/clone/cd12354-Movie-Picture-Pipeline/SETUP-GUIDE.md)
   - Step-by-step Terraform provisioning and IAM setup
   - GitHub Secrets configuration
   - Testing and deployment instructions
3. [CI-CD-PIPELINE.md](file:///c:/Users/cnyad/OneDrive/Desktop/clone/cd12354-Movie-Picture-Pipeline/CI-CD-PIPELINE.md)
   - Complete architectural walkthrough of all 4 workflows
   - Job dependencies, caching, and deployment mechanics
4. [PROJECT-SUMMARY.md](file:///c:/Users/cnyad/OneDrive/Desktop/clone/cd12354-Movie-Picture-Pipeline/PROJECT-SUMMARY.md)
   - Summary of project deliverables and infrastructure components

---

## 🗂️ Workflow Files (`.github/workflows/`)

| Workflow | File | Trigger | Key Jobs |
|---|---|---|---|
| Frontend CI | `frontend-ci.yml` | PR against `main` (`starter/frontend/**`) | `lint`, `test` (parallel) → `build` |
| Frontend CD | `frontend-cd.yml` | Push to `main` (`starter/frontend/**`) | `lint`, `test` → `build-and-push` → `deploy` |
| Backend CI | `backend-ci.yml` | PR against `main` (`starter/backend/**`) | `lint`, `test` (parallel) → `build` |
| Backend CD | `backend-cd.yml` | Push to `main` (`starter/backend/**`) | `lint`, `test` → `build-and-push` → `deploy` |