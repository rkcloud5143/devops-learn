# Docker, Terraform & CI/CD Q&A (200+ Questions)

Basic, Advanced, and Scenario-based questions.

---

# Docker (70 Questions)

## Basic

**Q1: What is Docker?**
A: Platform for building, shipping, and running containers. Packages applications with dependencies.

**Q2: What is a container?**
A: Isolated process with its own filesystem, networking, and process tree. Shares host kernel.

**Q3: What's the difference between a container and a VM?**
A: VMs virtualize hardware (full OS). Containers virtualize OS (share kernel). Containers are lighter and faster.

**Q4: What is a Docker image?**
A: Read-only template for creating containers. Built from Dockerfile. Layered filesystem.

**Q5: What is a Docker container?**
A: Running instance of an image. Has writable layer on top of image layers.

**Q6: What is a Dockerfile?**
A: Text file with instructions to build an image. FROM, RUN, COPY, CMD, etc.

**Q7: What is Docker Hub?**
A: Public registry for Docker images. Default registry.

**Q8: What is a Docker registry?**
A: Storage for Docker images. Docker Hub, ECR, GCR, private registries.

**Q9: What is the Docker daemon?**
A: Background service that manages containers. dockerd.

**Q10: What is the Docker CLI?**
A: Command-line interface to interact with Docker daemon.

**Q11: How do you run a container?**
A: `docker run image-name` or `docker run -d -p 80:80 nginx`

**Q12: What does `docker run -d` do?**
A: Runs container in detached mode (background).

**Q13: What does `docker run -p 8080:80` do?**
A: Maps host port 8080 to container port 80.

**Q14: How do you list running containers?**
A: `docker ps` or `docker container ls`

**Q15: How do you list all containers (including stopped)?**
A: `docker ps -a`

**Q16: How do you stop a container?**
A: `docker stop container-id`

**Q17: How do you remove a container?**
A: `docker rm container-id` or `docker rm -f container-id` (force)

**Q18: How do you list images?**
A: `docker images` or `docker image ls`

**Q19: How do you remove an image?**
A: `docker rmi image-id`

**Q20: How do you build an image?**
A: `docker build -t image-name:tag .`

**Q21: What is the build context?**
A: Files sent to Docker daemon for build. Usually current directory.

**Q22: What is .dockerignore?**
A: Excludes files from build context. Like .gitignore.

**Q23: How do you push an image to a registry?**
A: `docker push registry/image:tag`

**Q24: How do you pull an image?**
A: `docker pull image:tag`

**Q25: How do you exec into a running container?**
A: `docker exec -it container-id /bin/sh`

**Q26: How do you view container logs?**
A: `docker logs container-id` or `docker logs -f container-id` (follow)

**Q27: What is a Docker volume?**
A: Persistent storage for containers. Survives container removal.

**Q28: How do you create a volume?**
A: `docker volume create volume-name`

**Q29: How do you mount a volume?**
A: `docker run -v volume-name:/path/in/container image`

**Q30: What is a bind mount?**
A: Mounts host directory into container. `docker run -v /host/path:/container/path image`

---

## Dockerfile

**Q31: What does FROM do?**
A: Sets base image. First instruction in Dockerfile.

**Q32: What does RUN do?**
A: Executes command during build. Creates new layer.

**Q33: What does COPY do?**
A: Copies files from build context to image.

**Q34: What does ADD do?**
A: Like COPY but can extract archives and fetch URLs. Prefer COPY.

**Q35: What does CMD do?**
A: Default command when container starts. Can be overridden.

**Q36: What does ENTRYPOINT do?**
A: Main executable. CMD becomes arguments to ENTRYPOINT.

**Q37: What's the difference between CMD and ENTRYPOINT?**
A: CMD can be overridden completely. ENTRYPOINT is always run, CMD provides default arguments.

**Q38: What does EXPOSE do?**
A: Documents which ports the container listens on. Doesn't publish them.

**Q39: What does ENV do?**
A: Sets environment variables.

**Q40: What does WORKDIR do?**
A: Sets working directory for subsequent instructions.

**Q41: What does USER do?**
A: Sets user for subsequent instructions and container runtime.

**Q42: What does ARG do?**
A: Build-time variable. Passed with `--build-arg`.

**Q43: What is a multi-stage build?**
A: Multiple FROM statements. Copy artifacts between stages. Smaller final image.

**Q44: Why use multi-stage builds?**
A: Separate build dependencies from runtime. Smaller, more secure images.

**Q45: What is layer caching?**
A: Docker caches unchanged layers. Order instructions from least to most frequently changing.

---

## Advanced

**Q46: What is Docker Compose?**
A: Tool for defining multi-container applications. YAML file.

**Q47: How do you start services with Compose?**
A: `docker-compose up` or `docker-compose up -d`

**Q48: How do you stop Compose services?**
A: `docker-compose down` or `docker-compose down -v` (remove volumes)

**Q49: What is a Docker network?**
A: Isolated network for containers. Bridge, host, overlay, none.

**Q50: What is the bridge network?**
A: Default network. Containers can communicate by IP or container name.

**Q51: What is the host network?**
A: Container uses host's network directly. No isolation.

**Q52: What is an overlay network?**
A: Multi-host networking for Swarm/Kubernetes.

**Q53: How do you inspect a container?**
A: `docker inspect container-id`

**Q54: How do you see container resource usage?**
A: `docker stats`

**Q55: What is Docker Swarm?**
A: Docker's native orchestration. Simpler than Kubernetes.

**Q56: What is containerd?**
A: Container runtime. Docker uses it. Kubernetes can use it directly.

**Q57: What is the OCI?**
A: Open Container Initiative. Standards for container formats and runtimes.

**Q58: How do you limit container resources?**
A: `docker run --memory=512m --cpus=1 image`

**Q59: What is a Docker health check?**
A: Command to check container health. HEALTHCHECK instruction or --health-cmd.

**Q60: How do you clean up unused Docker resources?**
A: `docker system prune` or `docker system prune -a` (include unused images)

---

## Scenario-Based

**Q61: How do you reduce Docker image size?**
A: Multi-stage builds, smaller base images (alpine), minimize layers, remove unnecessary files, use .dockerignore.

**Q62: A container keeps restarting. How do you debug?**
A: `docker logs container-id`, `docker inspect container-id`, check exit code, exec into container if possible.

**Q63: How do you pass secrets to a container?**
A: Environment variables (not ideal), Docker secrets (Swarm), mount secret files, use external secret managers.

**Q64: How do you update a running container?**
A: You don't. Build new image, stop old container, start new one. Or use orchestration.

**Q65: Container can't connect to another container. What do you check?**
A: Same network? Correct hostname/IP? Port exposed? Firewall rules?

**Q66: How do you persist database data?**
A: Use volumes. `docker run -v db-data:/var/lib/mysql mysql`

**Q67: How do you run a container as non-root?**
A: USER instruction in Dockerfile. Or `docker run --user 1000:1000 image`

**Q68: How do you scan images for vulnerabilities?**
A: `docker scan image`, Trivy, Snyk, ECR scanning.

**Q69: How do you copy files from a container?**
A: `docker cp container-id:/path/file ./local`

**Q70: How do you see what changed in a container?**
A: `docker diff container-id`

---

# Terraform (70 Questions)

## Basic

**Q71: What is Terraform?**
A: Infrastructure as Code tool. Declarative. Manages cloud resources.

**Q72: What is Infrastructure as Code?**
A: Managing infrastructure through code files. Version controlled, repeatable, automated.

**Q73: What is HCL?**
A: HashiCorp Configuration Language. Terraform's syntax.

**Q74: What is a provider?**
A: Plugin that interacts with APIs. AWS, Azure, GCP, Kubernetes, etc.

**Q75: What is a resource?**
A: Infrastructure object managed by Terraform. EC2 instance, S3 bucket, etc.

**Q76: What is a data source?**
A: Fetches information from provider. Read-only. For referencing existing resources.

**Q77: What is terraform init?**
A: Initializes working directory. Downloads providers and modules.

**Q78: What is terraform plan?**
A: Shows what changes will be made. Dry run.

**Q79: What is terraform apply?**
A: Creates/updates infrastructure. Executes the plan.

**Q80: What is terraform destroy?**
A: Removes all managed infrastructure.

**Q81: What is the state file?**
A: Records what Terraform manages. Maps config to real resources. terraform.tfstate.

**Q82: Why is state important?**
A: Tracks resource IDs, dependencies, metadata. Required for updates and destroys.

**Q83: What is a variable?**
A: Input parameter. Defined in variables.tf, set via tfvars, env vars, or CLI.

**Q84: What is an output?**
A: Exposes values after apply. For use by other configs or users.

**Q85: What is a module?**
A: Reusable Terraform configuration. Container for multiple resources.

**Q86: What is the root module?**
A: The main working directory. Can call child modules.

**Q87: How do you reference a resource attribute?**
A: `resource_type.resource_name.attribute` e.g., `aws_instance.web.id`

**Q88: What is interpolation?**
A: Embedding expressions in strings. `"Hello, ${var.name}"`

**Q89: What is a local value?**
A: Named expression for reuse within a module. `locals { }` block.

**Q90: What is count?**
A: Creates multiple instances of a resource. `count = 3`

**Q91: What is for_each?**
A: Creates instances from a map or set. More flexible than count.

**Q92: What's the difference between count and for_each?**
A: count uses index (fragile if order changes). for_each uses keys (stable).

---

## Advanced

**Q93: What is remote state?**
A: State stored remotely (S3, Terraform Cloud). Enables team collaboration.

**Q94: How do you configure S3 backend?**
A:
```hcl
terraform {
  backend "s3" {
    bucket = "my-state"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```

**Q95: What is state locking?**
A: Prevents concurrent modifications. DynamoDB for S3 backend.

**Q96: What is terraform import?**
A: Brings existing resources under Terraform management.

**Q97: What is terraform taint?**
A: Marks resource for recreation on next apply. Deprecated, use -replace.

**Q98: What is terraform refresh?**
A: Updates state with real infrastructure. Now part of plan/apply.

**Q99: What are provisioners?**
A: Execute scripts on resources. local-exec, remote-exec. Use as last resort.

**Q100: Why avoid provisioners?**
A: Not declarative, can fail, hard to manage. Use user_data, config management instead.

**Q101: What is a null_resource?**
A: Resource that does nothing but can run provisioners. For arbitrary actions.

**Q102: What are workspaces?**
A: Separate state files in same config. For environments (dev, staging, prod).

**Q103: What is terraform fmt?**
A: Formats code to canonical style.

**Q104: What is terraform validate?**
A: Checks configuration syntax and consistency.

**Q105: What is terraform graph?**
A: Generates dependency graph in DOT format.

**Q106: What is the depends_on argument?**
A: Explicit dependency. When implicit dependencies aren't enough.

**Q107: What is lifecycle?**
A: Customizes resource behavior. create_before_destroy, prevent_destroy, ignore_changes.

**Q108: What is create_before_destroy?**
A: Creates replacement before destroying original. For zero-downtime.

**Q109: What is prevent_destroy?**
A: Prevents accidental deletion. Terraform errors if you try to destroy.

**Q110: What is ignore_changes?**
A: Ignores changes to specified attributes. For externally managed values.

**Q111: What is a dynamic block?**
A: Generates repeated nested blocks. For variable number of configurations.

**Q112: What are Terraform functions?**
A: Built-in functions for transformations. join, split, lookup, file, etc.

**Q113: What is the lookup function?**
A: Gets value from map with default. `lookup(map, key, default)`

**Q114: What is the file function?**
A: Reads file contents. `file("script.sh")`

**Q115: What is the templatefile function?**
A: Renders template with variables. `templatefile("template.tpl", { var = value })`

**Q116: What is sensitive = true?**
A: Marks variable/output as sensitive. Hidden in logs.

**Q117: What is terraform state mv?**
A: Moves resource in state. For refactoring without recreating.

**Q118: What is terraform state rm?**
A: Removes resource from state. Resource still exists, just unmanaged.

---

## Scenario-Based

**Q119: How do you manage multiple environments?**
A: Workspaces, separate directories, or separate tfvars files.

**Q120: How do you share state between configurations?**
A: terraform_remote_state data source. Or use outputs and data sources.

**Q121: Terraform plan shows changes you didn't make. Why?**
A: Drift. Someone changed resource outside Terraform. Or provider updated defaults.

**Q122: How do you handle secrets in Terraform?**
A: Don't commit to git. Use environment variables, Vault, AWS Secrets Manager, or encrypted tfvars.

**Q123: How do you upgrade provider versions safely?**
A: Pin versions, test in non-prod, review changelog, run plan before apply.

**Q124: Terraform apply failed midway. What now?**
A: State may be partially updated. Run plan to see current state. Fix issues and apply again.

**Q125: How do you refactor without destroying resources?**
A: terraform state mv to rename in state. Or import after changing config.

**Q126: How do you test Terraform code?**
A: terraform validate, terraform plan, Terratest, checkov, tflint.

**Q127: How do you structure a large Terraform project?**
A: Modules for reusable components, separate state per environment, consistent naming.

**Q128: How do you handle circular dependencies?**
A: Refactor to remove cycle. Use depends_on carefully. Split into separate applies if needed.

**Q129: How do you roll back a Terraform change?**
A: No built-in rollback. Revert code in git and apply. Or restore state backup.

**Q130: How do you prevent accidental destroys?**
A: prevent_destroy lifecycle, separate state for critical resources, require approval in CI/CD.

---

# CI/CD (60 Questions)

## Basic

**Q131: What is CI/CD?**
A: Continuous Integration / Continuous Delivery (or Deployment). Automates build, test, deploy.

**Q132: What is Continuous Integration?**
A: Frequently merging code changes. Automated build and test on every commit.

**Q133: What is Continuous Delivery?**
A: Code is always deployable. Automated pipeline to staging. Manual approval for prod.

**Q134: What is Continuous Deployment?**
A: Every change that passes tests is deployed automatically. No manual approval.

**Q135: What's the difference between Delivery and Deployment?**
A: Delivery = manual approval before prod. Deployment = fully automated to prod.

**Q136: What is a pipeline?**
A: Automated workflow. Stages: build, test, deploy.

**Q137: What is a build?**
A: Compiling code, creating artifacts. Docker image, JAR file, etc.

**Q138: What is an artifact?**
A: Output of build process. Deployable package.

**Q139: What is a stage?**
A: Group of jobs in a pipeline. Build stage, test stage, deploy stage.

**Q140: What is a job?**
A: Individual task in a pipeline. Run tests, build image, deploy.

**Q141: What are common CI/CD tools?**
A: Jenkins, GitHub Actions, GitLab CI, CircleCI, Travis CI, Azure DevOps, AWS CodePipeline.

**Q142: What is Jenkins?**
A: Open-source automation server. Highly customizable. Plugin ecosystem.

**Q143: What is a Jenkinsfile?**
A: Pipeline as code for Jenkins. Groovy syntax.

**Q144: What is GitHub Actions?**
A: CI/CD built into GitHub. Workflows defined in YAML.

**Q145: What is a GitHub Actions workflow?**
A: Automated process triggered by events. Defined in .github/workflows/.

**Q146: What is a GitHub Actions runner?**
A: Machine that executes jobs. GitHub-hosted or self-hosted.

**Q147: What is GitLab CI?**
A: CI/CD built into GitLab. .gitlab-ci.yml configuration.

**Q148: What is ArgoCD?**
A: GitOps continuous delivery for Kubernetes. Syncs cluster state with Git.

**Q149: What is GitOps?**
A: Git as single source of truth. Changes via pull requests. Automated sync.

**Q150: What is a webhook?**
A: HTTP callback triggered by events. Git push triggers CI pipeline.

---

## Advanced

**Q151: What is pipeline as code?**
A: Defining pipelines in version-controlled files. Jenkinsfile, .github/workflows/.

**Q152: What is a build matrix?**
A: Running same job with different parameters. Multiple OS, language versions.

**Q153: What is caching in CI/CD?**
A: Storing dependencies between runs. Speeds up builds.

**Q154: What are secrets in CI/CD?**
A: Sensitive values (API keys, passwords). Stored encrypted, injected at runtime.

**Q155: What is a deployment strategy?**
A: How you roll out changes. Rolling, blue-green, canary.

**Q156: What is rolling deployment?**
A: Gradually replace instances. Some old, some new during transition.

**Q157: What is blue-green deployment?**
A: Two identical environments. Switch traffic from blue to green.

**Q158: What is canary deployment?**
A: Deploy to small subset first. Monitor, then roll out to all.

**Q159: What is feature flagging?**
A: Toggle features without deploying. Decouple deployment from release.

**Q160: What is trunk-based development?**
A: Everyone commits to main branch. Short-lived feature branches. Requires good CI.

**Q161: What is GitFlow?**
A: Branching model. develop, feature, release, hotfix branches. More complex.

**Q162: What is a pull request / merge request?**
A: Request to merge branch. Code review, CI checks before merge.

**Q163: What is code review?**
A: Peer review of changes. Catches bugs, shares knowledge.

**Q164: What is static code analysis?**
A: Analyzing code without running it. Linters, security scanners.

**Q165: What is SAST?**
A: Static Application Security Testing. Finds vulnerabilities in source code.

**Q166: What is DAST?**
A: Dynamic Application Security Testing. Tests running application.

**Q167: What is SonarQube?**
A: Code quality and security platform. Static analysis, coverage, duplications.

**Q168: What is code coverage?**
A: Percentage of code executed by tests. Higher is generally better.

**Q169: What is a smoke test?**
A: Basic test that critical functionality works. Run after deployment.

**Q170: What is integration testing?**
A: Testing components together. API tests, database tests.

**Q171: What is end-to-end testing?**
A: Testing complete user flows. Simulates real user behavior.

**Q172: What is shift-left testing?**
A: Testing earlier in development. Find bugs sooner, cheaper to fix.

**Q173: What is infrastructure as code in CI/CD?**
A: Terraform/CloudFormation in pipeline. Automated infrastructure changes.

**Q174: What is immutable infrastructure?**
A: Never modify running servers. Replace with new ones. Consistent, reproducible.

**Q175: What is a deployment gate?**
A: Checkpoint requiring approval or conditions. Manual approval, test pass.

---

## Scenario-Based

**Q176: How do you handle failed deployments?**
A: Automated rollback, alerts, runbooks. Test in staging first.

**Q177: How do you implement zero-downtime deployments?**
A: Rolling updates, blue-green, health checks, connection draining.

**Q178: How do you secure CI/CD pipelines?**
A: Secrets management, least privilege, signed commits, audit logs, isolated runners.

**Q179: How do you speed up slow pipelines?**
A: Caching, parallelization, smaller tests, incremental builds, faster runners.

**Q180: How do you handle database migrations in CI/CD?**
A: Version migrations, run before deploy, backward compatible changes, rollback plan.

**Q181: How do you test infrastructure changes?**
A: terraform plan in PR, apply to test environment, automated tests, manual review.

**Q182: How do you manage multiple environments?**
A: Separate pipelines or stages, environment-specific configs, promotion between environments.

**Q183: How do you implement GitOps?**
A: Git repo for manifests, ArgoCD/Flux syncs to cluster, changes via PR.

**Q184: How do you handle hotfixes?**
A: Branch from prod, fix, test, deploy to prod, merge back to main.

**Q185: How do you ensure consistency across environments?**
A: Same artifacts, IaC, containerization, configuration management.

**Q186: A deployment succeeded but the app is broken. What do you do?**
A: Rollback immediately, investigate logs/metrics, fix and redeploy.

**Q187: How do you implement approval workflows?**
A: Required reviewers, deployment gates, manual approval steps.

**Q188: How do you handle long-running tests?**
A: Parallelize, run subset on PR, full suite nightly, prioritize critical tests.

**Q189: How do you version artifacts?**
A: Semantic versioning, git SHA, build number. Immutable once published.

**Q190: How do you handle secrets rotation?**
A: External secrets manager, automatic rotation, update references not values.

---

*Master these and you'll ace any DevOps interview! 🚀*
