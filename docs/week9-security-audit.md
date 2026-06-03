# Week 9 Security Audit — cis410-deploy-sa
**Project:** cis-410-cybersecurity-automation
**Date:** 2026-06-02
**Auditor:** Abduba Gabire

---

## 1. IAM Audit Results

### Before — Week 8 Configuration (over-permissioned)
| Role | Scope | Problem |
|---|---|---|
| roles/run.admin | Project | Overly broad — grants ability to delete services and modify IAM, not just deploy |
| roles/storage.admin | Project | Overly broad — grants access to ALL GCS buckets in the project |
| roles/artifactregistry.writer | Project | Acceptable — scoped to push images only |
| roles/viewer | Project | Acceptable — read-only project metadata |
| roles/iam.serviceAccountUser | Compute SA | Required — needed to act as Compute Engine default SA |

### After — Week 9 Least-Privilege Fix
| Role | Scope | Why Sufficient |
|---|---|---|
| roles/run.developer | Project | Deploy only — cannot delete services or modify IAM |
| roles/storage.admin | tfstate bucket only | Scoped to one bucket — not all storage |
| roles/artifactregistry.writer | Project | Unchanged — push images only |
| roles/viewer | Project | Unchanged — read project metadata |
| roles/iam.serviceAccountUser | Compute SA | Unchanged — required for Cloud Run deployment |

---

## 2. Secret Manager Migration
- **Secret created:** `flask-app-secret`
- **Replication:** automatic
- **Access granted to:** `cis410-deploy-sa` — roles/secretmanager.secretAccessor on this secret only
- **Access granted to:** `PROJECT_NUMBER-compute@developer.gserviceaccount.com` — roles/secretmanager.secretAccessor on this secret only (required for Cloud Run runtime access)
- **Cloud Run update:** APP_SECRET environment variable mounted from Secret Manager at runtime

---

## 3. Monitoring Configuration
- **Log-based alert:** `cis410-flask-app-alert` — fires on severity>=WARNING for cis410-flask-app
- **Notification channel:** <!-- your student email -->
- **Billing budget:** `cis410-monthly-budget` — $20 limit, alerts at 50% / 90% / 100%

---

## 4. Reflection

**Q1: Why is roles/run.admin inappropriate for a CI/CD pipeline service account?**
the reason why roles/run.admin is inappropriate for a CI/CD pipeline service account is because a CI/CD account only needs to deploy revisions it should not be able to delete, roll back traffic or modify IAM policies. if Roles/run.admin is granted it will alot it to delete and roll back traffic meaning something like a compromised token can be used to take down the entire application or escalate privileges. the least privilege principal says the SA should only hold roles/run.developer which covers deployment without the destruction permissions.

---

**Q2: What is the security difference between storing a secret in GitHub Secrets vs. Google Secret Manager?**
The difference between the two is GitHub Secrets are like lockers only the pipeline can open. The secrets are encrypted at rest and injected as environment variables during CI/CD runs — the secret is used during deployment but never travels to the live app. Google Secret Manager is like a secure bank vault the live app itself can open at any time, but the secret never touches your code or pipeline because Google Secret Manager stores secrets independently of your code infrastructure, giving you greater control with fine-grained IAM access per secret and can be accessed at runtime by Cloud Run. This means the secret never has to be baked into a container image or passed through a pipeline at all. There is also a log of every time someone 
opens it, showing exactly who or what accessed a secret and when.

---

**Q3: A coworker says "I will clean up IAM permissions after the project launches. For now I need everything to work fast." What is the risk of this approach?**
The risk with this approach is that people often rush things, and in this case it is an easy fix that should have been completed immediately. When teams are focused on new features or incidents post-launch, cleanup tasks will no longer be the main priority and will get pushed aside indefinitely. An overly permissioned service account becomes an attack surface that bad actors can take advantage of;  if those credentials are leaked or a dependency is compromised, the blast radius is much larger than it needed to be. This is why IAM permissions should be properly scoped before a project launches, not after.