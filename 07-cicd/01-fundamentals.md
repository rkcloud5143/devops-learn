# CI/CD — Fundamentals

## What is CI/CD?
```
Developer pushes code
        │
        ▼
┌─── CI (Continuous Integration) ──────────────────┐
│                                                    │
│  1. Pull code from GitHub                         │
│  2. Install dependencies                          │
│  3. Run tests                                     │
│  4. Build Docker image                            │
│  5. Push image to ECR                             │
│                                                    │
└────────────────────┬─────────────────────────────┘
                     │ (if all tests pass)
                     ▼
┌─── CD (Continuous Deployment) ───────────────────┐
│                                                    │
│  6. Deploy to staging                             │
│  7. Run smoke tests                               │
│  8. Deploy to production                          │
│                                                    │
└──────────────────────────────────────────────────┘
```
