# Jenkins — Complete Guide 🏗️

The most widely used CI/CD tool in enterprise environments.

---

## What is Jenkins?

```
Open-source automation server.
Runs on a server you manage (EC2, on-prem, Kubernetes).
Highly customizable with 1800+ plugins.

┌─── Jenkins Architecture ──────────────────────────────┐
│                                                        │
│  ┌─── Controller (Master) ──────────────────────────┐ │
│  │  - Manages pipelines                              │ │
│  │  - Schedules jobs                                 │ │
│  │  - Serves web UI                                  │ │
│  │  - Stores configuration                           │ │
│  └──────────────────────────────────────────────────┘ │
│           │              │              │              │
│           ▼              ▼              ▼              │
│  ┌─── Agent 1 ──┐ ┌─── Agent 2 ──┐ ┌─── Agent 3 ──┐ │
│  │ Runs builds   │ │ Runs builds   │ │ Runs builds   │ │
│  │ (EC2/Docker)  │ │ (EC2/Docker)  │ │ (K8s pod)     │ │
│  └───────────────┘ └───────────────┘ └───────────────┘ │
└────────────────────────────────────────────────────────┘
```

---

## Jenkinsfile (Pipeline as Code)

Jenkins pipelines are defined in a `Jenkinsfile` at the root of your repo.

### Declarative Pipeline (Recommended)

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        AWS_REGION     = 'ca-central-1'
        ECR_REPO       = '123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app'
        EKS_CLUSTER    = 'my-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${ECR_REPO}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageTag} ."
                    env.IMAGE_TAG = imageTag
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: AWS_REGION) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $ECR_REPO
                        docker push $IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            when {
                branch 'main'
            }
            steps {
                withAWS(credentials: 'aws-credentials', region: AWS_REGION) {
                    sh """
                        aws eks update-kubeconfig --name $EKS_CLUSTER
                        helm upgrade --install my-app ./helm/my-app \
                            --set image.tag=${env.BUILD_NUMBER} \
                            --namespace production \
                            --wait
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend(color: 'good', message: "✅ Build #${env.BUILD_NUMBER} succeeded")
        }
        failure {
            slackSend(color: 'danger', message: "❌ Build #${env.BUILD_NUMBER} failed")
        }
        always {
            cleanWs()  // Clean workspace
        }
    }
}
```

---

## Key Concepts

### Agents
```groovy
// Run on any available agent
agent any

// Run on agent with specific label
agent { label 'linux' }

// Run in Docker container
agent {
    docker {
        image 'node:20'
    }
}

// Run in Kubernetes pod
agent {
    kubernetes {
        yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:dind
            securityContext:
              privileged: true
        '''
    }
}
```

### Credentials
```groovy
// Stored in Jenkins → Manage Jenkins → Credentials
// Types: Username/Password, Secret text, SSH key, AWS credentials

// Use in pipeline
withCredentials([
    usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')
]) {
    sh "docker login -u $USER -p $PASS"
}

// AWS credentials
withAWS(credentials: 'aws-creds', region: 'ca-central-1') {
    sh 'aws s3 ls'
}
```

### Parameters (Manual Input)
```groovy
pipeline {
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deploy to')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip tests?')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
    }
    stages {
        stage('Deploy') {
            steps {
                sh "helm upgrade --install my-app ./chart --set env=${params.ENVIRONMENT}"
            }
        }
    }
}
```

### Parallel Stages
```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'npm run test:unit' }
        }
        stage('Integration Tests') {
            steps { sh 'npm run test:integration' }
        }
        stage('Security Scan') {
            steps { sh 'trivy fs .' }
        }
    }
}
```

### Shared Libraries (Reusable Code)
```groovy
// vars/deployToEKS.groovy (in shared library repo)
def call(Map config) {
    sh """
        aws eks update-kubeconfig --name ${config.cluster}
        helm upgrade --install ${config.app} ./chart \
            --set image.tag=${config.tag} \
            --namespace ${config.namespace}
    """
}

// Jenkinsfile (usage)
@Library('my-shared-lib') _

pipeline {
    stages {
        stage('Deploy') {
            steps {
                deployToEKS(
                    cluster: 'my-cluster',
                    app: 'my-app',
                    tag: env.BUILD_NUMBER,
                    namespace: 'production'
                )
            }
        }
    }
}
```

---

## Jenkins on Kubernetes

Running Jenkins itself on EKS:

```yaml
# Install Jenkins on K8s with Helm
helm repo add jenkins https://charts.jenkins.io
helm install jenkins jenkins/jenkins \
    --namespace jenkins \
    --set controller.serviceType=LoadBalancer \
    --set agent.enabled=true
```

Benefits:
- Agents spin up as pods → scale automatically
- No permanent agent servers to maintain
- Each build gets a fresh, clean environment

---

## Jenkins Best Practices

1. **Pipeline as code** — Always use Jenkinsfile, never click-based jobs
2. **Shared libraries** — Reuse common pipeline logic
3. **Credentials plugin** — Never hardcode secrets
4. **Agent labels** — Route jobs to appropriate agents
5. **Parallel stages** — Speed up pipelines
6. **Post actions** — Always clean up, always notify
7. **Blue Ocean UI** — Better visualization (install plugin)
8. **Backup** — Backup Jenkins home directory regularly
9. **Security** — RBAC, LDAP/SSO integration, audit trail
10. **Kubernetes agents** — Dynamic, scalable, clean

---

*Jenkins is powerful but needs maintenance. Great for enterprise, complex workflows! 🏗️*
