# Jenkins — In-Depth Guide 🏗️

Architecture, scaling, security, migration to GitHub Actions, and real-world patterns.

---

## Jenkins Architecture Deep Dive

```
┌─── Jenkins Controller ─────────────────────────────────────┐
│                                                             │
│  • Manages UI, configuration, job scheduling                │
│  • Stores job definitions, build history, credentials       │
│  • NEVER run builds on controller (security + stability)    │
│  • Single point of failure — back it up!                    │
│                                                             │
│  JENKINS_HOME (/var/lib/jenkins):                           │
│    config.xml          → Global settings                    │
│    jobs/               → Job definitions                    │
│    workspace/          → Build workspaces                   │
│    plugins/            → Installed plugins                  │
│    credentials.xml     → Encrypted credentials             │
│    secrets/            → Master encryption key              │
└─────────────────────────────────────────────────────────────┘
         │
         │ Dispatches builds to agents
         │
┌─── Agents (Where builds actually run) ─────────────────────┐
│                                                             │
│  Static Agents:                                             │
│    • EC2 instances always running                           │
│    • Connected via SSH or JNLP                              │
│    • Wasteful if idle                                       │
│                                                             │
│  Dynamic Agents (Kubernetes):                               │
│    • Pod spins up → runs build → dies                       │
│    • Scale to zero when idle                                │
│    • Fresh environment every build                          │
│    • Docker-in-Docker or kaniko for image builds            │
│                                                             │
│  Dynamic Agents (EC2 Plugin):                               │
│    • Launches EC2 on demand                                 │
│    • Terminates after idle timeout                          │
│    • Good for heavy builds (needs big instance)             │
└─────────────────────────────────────────────────────────────┘
```

---

## Jenkins on Kubernetes (EKS)

```yaml
# Deployed via Helm + ArgoCD
# Controller on managed node group (stable)
# Agents on Karpenter nodes (auto-scale)

# Agent pod template (defined in Jenkinsfile or config):
podTemplate:
  containers:
    - name: jnlp                    # Required: connects to controller
      image: jenkins/inbound-agent
    - name: docker                  # For docker build
      image: docker:dind
      privileged: true              # Required for DinD
    - name: terraform               # For IaC
      image: hashicorp/terraform
    - name: kubectl                 # For K8s deploys
      image: bitnami/kubectl
```

### Custom Agent Images
```dockerfile
# DevOps agent with all tools pre-installed
FROM jenkins/inbound-agent:latest

USER root
RUN apt-get update && apt-get install -y \
    docker.io \
    awscli \
    kubectl \
    helm \
    terraform \
    terragrunt \
    trivy \
    trufflehog \
    hadolint \
    shellcheck \
    yamllint \
    jq yq \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

---

## Jenkinsfile Patterns

### Multi-Branch Pipeline (Most Common)
```groovy
// Automatically discovers branches and PRs
// Jenkinsfile at repo root — different behavior per branch

pipeline {
    agent { kubernetes { yaml agentPodYaml() } }
    
    environment {
        ECR_REPO = credentials('ecr-repo-url')
    }
    
    stages {
        stage('Validate') {
            steps {
                sh 'make lint'        // yamllint, shellcheck, hadolint
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'trufflehog git file://. --only-verified'
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }
        
        stage('Build') {
            when { branch 'main' }    // Only build image on main
            steps {
                sh '''
                    aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPO
                    docker build -t $ECR_REPO:${GIT_COMMIT} .
                    docker push $ECR_REPO:${GIT_COMMIT}
                '''
            }
        }
        
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh '''
                    # Update GitOps repo (ArgoCD picks up the change)
                    yq -i ".images.myapp = \"${GIT_COMMIT}\"" gitops-repo/images.yaml
                    cd gitops-repo && git add . && git commit -m "Deploy myapp:${GIT_COMMIT}" && git push
                '''
            }
        }
    }
    
    post {
        failure {
            slackSend(color: 'danger', message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed")
        }
        success {
            slackSend(color: 'good', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} passed")
        }
    }
}
```

### Terraform Pipeline
```groovy
pipeline {
    agent { kubernetes { yaml terraformAgent() } }
    
    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Environment')
        choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Action')
    }
    
    stages {
        stage('Init') {
            steps {
                dir("environments/${params.ENV}") {
                    sh 'terragrunt run-all init'
                }
            }
        }
        
        stage('Plan') {
            steps {
                dir("environments/${params.ENV}") {
                    sh 'terragrunt run-all plan --terragrunt-non-interactive 2>&1 | tee plan.txt'
                }
            }
        }
        
        stage('Approve') {
            when { expression { params.ACTION == 'apply' && params.ENV == 'prod' } }
            steps {
                input message: "Apply to PRODUCTION?", ok: 'Yes, apply'
            }
        }
        
        stage('Apply') {
            when { expression { params.ACTION == 'apply' } }
            steps {
                dir("environments/${params.ENV}") {
                    sh 'terragrunt run-all apply --terragrunt-non-interactive'
                }
            }
        }
    }
}
```

---

## Jenkins Security

```
✅ Best practices:
  • Run controller on private network (not internet-facing)
  • LDAP/SSO integration (not local accounts)
  • Role-Based Access Control (Matrix Auth plugin)
  • Credential plugin for ALL secrets (never in Jenkinsfile)
  • Audit trail plugin (who did what)
  • Disable script console for non-admins
  • Agents in separate security group from controller
  • Pin plugin versions (auto-update can break things)

✅ Credential types:
  • Username + Password
  • Secret text (API tokens)
  • SSH private key
  • AWS credentials (access key + secret)
  • Kubernetes service account
  • Certificate (PKCS#12)
  
  Access in Jenkinsfile:
    environment {
        AWS_CREDS = credentials('aws-credentials')  // Injects _USR and _PSW
        API_TOKEN = credentials('api-token')        // Injects as env var
    }
```

---

## Jenkins → GitHub Actions Migration

```
Why migrate?
  • No infrastructure to maintain
  • Native GitHub integration (PR checks, branch protection)
  • OIDC auth (no stored AWS keys)
  • Matrix builds built-in
  • Marketplace of pre-built actions
  • Simpler YAML vs Groovy

Migration mapping:
  Jenkins                    → GitHub Actions
  ─────────────────          ─────────────────────
  Jenkinsfile                → .github/workflows/*.yml
  pipeline { }               → jobs: { }
  stages { }                 → steps: [ ]
  agent { }                  → runs-on: / container:
  when { branch 'main' }    → if: github.ref == 'refs/heads/main'
  parameters { }             → workflow_dispatch: inputs:
  credentials()              → secrets.NAME
  post { }                   → jobs.<id>.steps (always/failure conditions)
  shared libraries           → reusable workflows / composite actions
  plugins                    → actions (marketplace)

Keep Jenkins for:
  • Complex orchestration (multiple repos, approval chains)
  • Legacy jobs too complex to migrate
  • On-prem requirements
```

---

*Jenkins isn't dead — but know when to use it vs GitHub Actions! 🏗️*
