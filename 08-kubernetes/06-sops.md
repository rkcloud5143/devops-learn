# SOPS — Secrets Management for GitOps 🔐

Encrypt secrets in Git so you can safely store them alongside your code.

---

## The Problem

```
You want to store Kubernetes secrets in Git (for GitOps/ArgoCD).
But secrets in plain text in Git = DISASTER.

❌ Bad: secrets.yaml in Git
apiVersion: v1
kind: Secret
data:
  DB_PASSWORD: bXlfc3VwZXJfc2VjcmV0    ← base64 is NOT encryption!

Anyone with repo access can decode: echo "bXlfc3VwZXJfc2VjcmV0" | base64 -d
→ my_super_secret 😱
```

---

## What is SOPS?

```
SOPS (Secrets OPerationS) by Mozilla.
Encrypts ONLY the values in YAML/JSON files, not the keys.
You can still see the structure, but values are encrypted.

✅ Good: secrets.yaml encrypted with SOPS
apiVersion: v1
kind: Secret
data:
    DB_PASSWORD: ENC[AES256_GCM,data:abc123...,type:str]    ← Encrypted!

Keys are readable (you know what's there).
Values are encrypted (you can't read them without the key).
```

**Pizza shop:** The recipe book is on the shelf for everyone to see the recipe names. But the secret sauce ingredients are written in a code only the head chef can decode.

---

## How SOPS Works

```
┌─── Your Machine ──────────────────────────────────────────┐
│                                                            │
│  secrets.yaml (plain text)                                │
│       │                                                    │
│       ▼                                                    │
│  sops --encrypt secrets.yaml                              │
│       │                                                    │
│       ▼                                                    │
│  secrets.enc.yaml (encrypted)  ──→  Push to Git ✅        │
│                                                            │
└────────────────────────────────────────────────────────────┘

┌─── ArgoCD / CI/CD ────────────────────────────────────────┐
│                                                            │
│  Pull secrets.enc.yaml from Git                           │
│       │                                                    │
│       ▼                                                    │
│  sops --decrypt secrets.enc.yaml                          │
│       │  (uses KMS key — needs IAM permissions)           │
│       ▼                                                    │
│  secrets.yaml (plain text) ──→ Apply to Kubernetes        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## SOPS with AWS KMS

### Step 1: Create a KMS Key

```bash
aws kms create-key --description "SOPS encryption key"
# Note the KeyId: arn:aws:kms:ca-central-1:123456789012:key/abc-123-def

# Create an alias (easier to remember)
aws kms create-alias \
    --alias-name alias/sops-key \
    --target-key-id abc-123-def
```

### Step 2: Create .sops.yaml Config

```yaml
# .sops.yaml (in repo root)
creation_rules:
  # Encrypt all files matching this pattern with this KMS key
  - path_regex: .*\.enc\.yaml$
    kms: arn:aws:kms:ca-central-1:123456789012:key/abc-123-def

  # Different key per environment
  - path_regex: dev/.*\.enc\.yaml$
    kms: arn:aws:kms:ca-central-1:111111111111:key/dev-key-id

  - path_regex: prod/.*\.enc\.yaml$
    kms: arn:aws:kms:ca-central-1:222222222222:key/prod-key-id
```

### Step 3: Encrypt a File

```bash
# Install SOPS
brew install sops

# Create your secret file
cat > secrets.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secrets
  namespace: production
type: Opaque
stringData:
  DB_PASSWORD: my_super_secret
  API_KEY: sk-abc123def456
EOF

# Encrypt it
sops --encrypt secrets.yaml > secrets.enc.yaml

# Now secrets.enc.yaml looks like:
# apiVersion: v1
# kind: Secret
# metadata:
#     name: my-app-secrets
#     namespace: production
# type: Opaque
# stringData:
#     DB_PASSWORD: ENC[AES256_GCM,data:8dK3mN...,iv:abc...,tag:def...]
#     API_KEY: ENC[AES256_GCM,data:pQ7rS...,iv:ghi...,tag:jkl...]
# sops:
#     kms:
#         - arn: arn:aws:kms:ca-central-1:123456789012:key/abc-123-def
#     version: 3.8.1

# Delete the plain text file!
rm secrets.yaml

# Commit encrypted file to Git
git add secrets.enc.yaml .sops.yaml
git commit -m "Add encrypted secrets"
```

### Step 4: Decrypt (When Needed)

```bash
# Decrypt to stdout (view)
sops --decrypt secrets.enc.yaml

# Decrypt to file
sops --decrypt secrets.enc.yaml > secrets.yaml

# Edit in place (decrypts, opens editor, re-encrypts on save)
sops secrets.enc.yaml
```

---

## SOPS + ArgoCD

### Option 1: KSOPS Plugin (Recommended)

```yaml
# Install KSOPS (Kustomize plugin for SOPS)
# ArgoCD needs the SOPS plugin to decrypt during sync

# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

generators:
  - secret-generator.yaml

# secret-generator.yaml
apiVersion: viaduct.ai/v1
kind: ksops
metadata:
  name: my-secrets
files:
  - secrets.enc.yaml
```

### Option 2: Helm Secrets Plugin

```bash
# Install helm-secrets plugin
helm plugin install https://github.com/jkroepke/helm-secrets

# Encrypt values file
sops --encrypt values-secrets.yaml > values-secrets.enc.yaml

# Use with Helm
helm secrets upgrade my-app ./chart \
    -f values.yaml \
    -f values-secrets.enc.yaml
```

---

## SOPS Commands

```bash
# Encrypt
sops --encrypt file.yaml > file.enc.yaml
sops --encrypt --in-place file.yaml          # Encrypt in place

# Decrypt
sops --decrypt file.enc.yaml                 # To stdout
sops --decrypt --in-place file.enc.yaml      # In place

# Edit (decrypt → edit → re-encrypt)
sops file.enc.yaml

# Rotate keys (re-encrypt with new key)
sops --rotate --in-place file.enc.yaml

# Update keys (after changing .sops.yaml)
sops updatekeys file.enc.yaml
```

---

## SOPS vs Other Secret Solutions

| Tool | How it works | Best for |
|------|-------------|----------|
| **SOPS** | Encrypts files in Git | GitOps, small teams |
| **External Secrets Operator** | Syncs from Secrets Manager/Vault to K8s | Large teams, centralized secrets |
| **Sealed Secrets** | Encrypts with cluster-specific key | Single cluster |
| **HashiCorp Vault** | Centralized secret store | Enterprise, dynamic secrets |
| **AWS Secrets Manager** | AWS-managed secret store | AWS-native apps |

### When to Use SOPS
- You want secrets in Git (GitOps)
- Small to medium team
- Using ArgoCD or Flux
- Want simple encryption with KMS

### When NOT to Use SOPS
- Hundreds of secrets (use Vault or Secrets Manager)
- Need dynamic secrets (use Vault)
- Need automatic rotation (use Secrets Manager)

---

## Security Best Practices

1. **Never commit plain text secrets** — Always encrypt first
2. **Use KMS** — Not PGP (KMS is managed, rotatable, auditable)
3. **Separate keys per environment** — Dev key can't decrypt prod secrets
4. **IAM controls** — Only CI/CD and ArgoCD roles can use the KMS key
5. **Git hooks** — Pre-commit hook to prevent pushing unencrypted secrets
6. **.gitignore plain text** — `*.decrypted.yaml` in .gitignore
7. **Rotate keys** — Periodically rotate KMS keys

### Pre-commit Hook (Prevent Accidents)
```bash
#!/bin/bash
# .git/hooks/pre-commit
# Prevent committing files with unencrypted secrets

for file in $(git diff --cached --name-only | grep -E '\.yaml$' | grep -v '\.enc\.yaml$'); do
    if grep -q 'stringData:' "$file" || grep -q 'password' "$file"; then
        echo "ERROR: $file may contain unencrypted secrets!"
        echo "Encrypt with: sops --encrypt $file > ${file%.yaml}.enc.yaml"
        exit 1
    fi
done
```

---

*SOPS = secrets in Git, safely. Perfect for GitOps! 🔐*
