# Git — Pull Request Workflow

## How Real Teams Work
```
main ─────────────────────────────────────────►
       │                              ▲
       │ git checkout -b feature      │ merge PR
       ▼                              │
feature ──► commit ──► push ──► Open PR on GitHub
                                  │
                                  ├── Code review
                                  ├── CI/CD runs tests
                                  └── Approve & merge
```

## Steps
1. Create a branch from `main`
2. Make changes, commit, push to GitHub
3. Open a Pull Request on GitHub
4. Team reviews the code
5. CI/CD pipeline runs automated tests
6. Reviewer approves → merge into `main`
7. Delete the feature branch
