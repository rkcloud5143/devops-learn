# HashiCorp Vault — Enterprise Secrets Management 🔐

Centralized secrets for when SOPS and Secrets Manager aren't enough.

---

## What is Vault?

```
AWS Secrets Manager = Store and rotate secrets (AWS-only)
SOPS                = Encrypt secrets in Git files
Vault               = Full secrets PLATFORM (any cloud, dynamic secrets, PKI, encryption)

┌─── Vault ──────────────────────────────────────────────┐
│                                                         │
│  Static Secrets    → Store passwords, API keys          │
│  Dynamic Secrets   → Generate temporary DB credentials  │
│  Encryption        → Encrypt/decrypt data via API       │
│  PKI               → Issue TLS certificates             │
│  Identity          → Authenticate apps and users        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## How It Works

```
App needs DB password:

Without Vault:
  App reads password from env var / config file
  Password is static, never rotates
  If leaked, attacker has permanent access

With Vault (Dynamic Secrets):
  App authenticates to Vault (via IAM role, K8s SA, etc.)
  Vault creates a TEMPORARY DB user/password (TTL: 1 hour)
  App uses temporary credentials
  Vault auto-revokes after TTL expires
  If leaked, credentials expire automatically ✅
```

---

## Key Concepts

```
Secret Engine    — Backend that stores/generates secrets
                   (KV, AWS, database, PKI, transit)

Auth Method      — How clients authenticate to Vault
                   (Token, AWS IAM, Kubernetes, LDAP, OIDC)

Policy           — What a client can access
                   (path "secret/data/prod/*" { capabilities = ["read"] })

Lease            — TTL on dynamic secrets (auto-revoke)

Seal/Unseal      — Vault starts sealed (encrypted). Must unseal to use.
```

---

## Common Operations

```bash
# ─── Start / Status ───
vault status
vault operator init                    # Initialize (first time)
vault operator unseal                  # Unseal with key shares

# ─── KV Secrets (Static) ───
vault kv put secret/myapp/prod db_password=s3cret api_key=abc123
vault kv get secret/myapp/prod
vault kv get -field=db_password secret/myapp/prod
vault kv list secret/myapp/
vault kv delete secret/myapp/prod

# ─── Dynamic Secrets (Database) ───
# Vault creates temporary DB credentials on demand
vault read database/creds/my-role
# Returns: username=v-token-my-role-abc123, password=xyz789, ttl=1h

# ─── AWS Dynamic Credentials ───
vault read aws/creds/my-role
# Returns: temporary access_key + secret_key (auto-revoked)

# ─── Encryption as a Service ───
vault write transit/encrypt/my-key plaintext=$(echo "secret" | base64)
vault write transit/decrypt/my-key ciphertext=vault:v1:abc123...

# ─── Policies ───
vault policy write my-policy - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}
EOF
```

---

## Vault + Kubernetes

```yaml
# External Secrets Operator — syncs Vault secrets to K8s Secrets
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: my-app-secrets
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: secret/data/myapp/prod
        property: db_password
```

```bash
# Vault Agent Sidecar (alternative)
# Injects secrets directly into pod filesystem
# Annotate pod:
#   vault.hashicorp.com/agent-inject: "true"
#   vault.hashicorp.com/role: "my-app"
#   vault.hashicorp.com/agent-inject-secret-db: "secret/data/myapp/prod"
```

---

## When to Use What

| Tool | Best For |
|------|----------|
| **AWS Secrets Manager** | AWS-only, simple rotation, RDS integration |
| **SSM Parameter Store** | Simple config + secrets, free tier |
| **SOPS** | Encrypting files in Git (GitOps) |
| **Sealed Secrets** | Single K8s cluster, simple |
| **Vault** | Multi-cloud, dynamic secrets, PKI, enterprise compliance |

---

*Vault = the enterprise-grade secrets platform. Overkill for small teams, essential for large orgs! 🔐*
