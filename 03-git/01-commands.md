# Git — Essential Commands

## Git Workflow Diagram
```
Working Directory ──► Staging Area ──► Local Repo ──► Remote (GitHub)
     (edit files)    (git add)       (git commit)    (git push)

                  ◄──────────────────────────────── (git pull)
                                                    (git clone)
```

## Setup
```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

## Daily Workflow
```bash
git clone <url>           # Download a repo
git status                # What changed?
git add .                 # Stage all changes
git commit -m "message"   # Save changes locally
git push                  # Upload to GitHub
git pull                  # Download latest changes
```

## Branching (How Teams Work)
```bash
git branch feature-x      # Create branch
git checkout feature-x    # Switch to branch
git checkout -b feature-x # Create + switch (shortcut)
git merge feature-x       # Merge branch into current
git branch -d feature-x   # Delete branch after merge
```

## Undo Mistakes
```bash
git stash                 # Temporarily save changes
git stash pop             # Restore stashed changes
git log --oneline         # View commit history
git reset --hard HEAD~1   # Undo last commit (careful!)
```

## Checklist
- [ ] Create a GitHub account, make 5+ commits
- [ ] Create a branch, open a PR, merge it
