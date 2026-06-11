# 🚀 Complete DevOps Interview Questions & Answers
> Compiled from Days 15, 16, 17 & 21 of Abhishek's Complete DevOps Course
> Topics: Ansible · Infrastructure as Code · Terraform · Jenkins · CI/CD

---

## 📚 Table of Contents

1. [Ansible Interview Questions](#1-ansible-interview-questions)
2. [Infrastructure as Code (IaC) Interview Questions](#2-infrastructure-as-code-iac-interview-questions)
3. [Terraform Interview Questions](#3-terraform-interview-questions)
4. [Jenkins & CI/CD Interview Questions](#4-jenkins--cicd-interview-questions)
5. [Mixed / Cross-Topic Questions](#5-mixed--cross-topic-questions)
6. [Scenario-Based Questions (Senior Level)](#6-scenario-based-questions-senior-level)

---

## 1. Ansible Interview Questions

### 🟢 Beginner

---

**Q1. What is Ansible and why is it used?**

> Ansible is an open-source, agentless configuration management tool that automates software provisioning, application deployment, and configuration management. It uses SSH to communicate with target servers, requires no agent installation on targets, and is written in Python. It is used because it is simple (YAML-based), agentless, and works on any OS.

---

**Q2. What is the difference between Ansible ad-hoc commands and Ansible Playbooks?**

> Ad-hoc commands are one-liners executed directly from the CLI for simple, one-off tasks — like creating a file, checking disk space, or running a quick command on target servers. Playbooks are YAML files that define a sequence of multiple tasks to be executed in order. Use ad-hoc for 1–2 tasks; use Playbooks for complex, repeatable automation workflows.

```bash
# Ad-hoc example:
ansible -i inventory all -m shell -a "df -h"

# Playbook example:
ansible-playbook -i inventory site.yml
```

---

**Q3. What is passwordless authentication in Ansible and how do you set it up?**

> Ansible requires SSH-based passwordless authentication to connect to target servers without prompting for a password. Setup steps:
> 1. Run `ssh-keygen` on the Ansible control node to generate a public/private key pair
> 2. Copy the public key (`~/.ssh/id_rsa.pub`) content
> 3. Paste it into `~/.ssh/authorized_keys` on every target server
> 4. Test with `ssh <target-ip>` — it should connect without a password prompt

---

**Q4. What is an Ansible inventory file?**

> An inventory file lists the IP addresses or hostnames of all target servers that Ansible manages. It supports grouping servers so you can run playbooks on specific sets of servers.

```ini
# Simple inventory:
172.31.62.28

# Grouped inventory:
[webservers]
172.31.62.100
172.31.62.101

[dbservers]
172.31.62.200
```

---

**Q5. What does `become: true` do in an Ansible Playbook?**

> It escalates privileges to run tasks as the root user (equivalent to `sudo`). Required for tasks like installing packages (`apt install`) or managing system services that need elevated permissions.

---

### 🟡 Intermediate

---

**Q6. How do you group servers in Ansible and run a playbook only on a specific group?**

> Use bracket notation in the inventory file to define groups. Then pass the group name instead of `all` when running the playbook.

```bash
ansible-playbook -i inventory webservers playbook.yml
# Only targets servers under [webservers]
```

---

**Q7. What is Gather Facts in Ansible and can it be disabled?**

> Gather Facts is an automatic first step in every playbook where Ansible collects information about the target server — OS type, IP addresses, memory, CPU, etc. This info can be used as variables in tasks. It can be disabled with `gather_facts: false` to speed up playbooks where system info isn't needed.

---

**Q8. What is the difference between `shell` module and `apt` module in Ansible?**

> Both can install packages, but the `apt` module is purpose-built for package management on Debian/Ubuntu — it's idempotent (safe to run multiple times), handles edge cases, and is more reliable. The `shell` module just runs raw shell commands — less idempotent, more prone to errors.

```yaml
# Using apt module (preferred):
- name: Install nginx
  apt:
    name: nginx
    state: present

# Using shell (avoid if a dedicated module exists):
- name: Install nginx
  shell: apt install nginx -y
```

---

**Q9. What are Ansible Roles and why are they used?**

> Roles are a structured way to organize complex playbooks into reusable, modular components. They are used when a playbook grows to 50+ tasks with variables, files, templates, and handlers — keeping everything in one file becomes unmanageable. Roles follow a standard folder structure.

```
my-role/
├── tasks/       # Main task logic
├── handlers/    # Event-triggered tasks (e.g., restart service)
├── vars/        # Variables (higher priority)
├── defaults/    # Default variable values (overridable)
├── files/       # Static files to copy to targets
├── templates/   # Jinja2 dynamic templates
├── meta/        # Metadata, dependencies, license
└── tests/       # Unit tests
```

---

**Q10. What is the purpose of Handlers in Ansible?**

> Handlers are tasks triggered only when notified by another task — typically used for actions that should only run when a change occurs. For example: restart nginx only if the config file changed. Without handlers, you would restart nginx on every playbook run even if nothing changed.

```yaml
tasks:
  - name: Copy nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx       # Only triggers if file changed

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

---

**Q11. What is the difference between `vars/` and `defaults/` in an Ansible Role?**

> Both store variables but differ in priority. `vars/` has higher priority and is harder to override — used for role-internal variables that should not change. `defaults/` has the lowest priority and is easily overridden by inventory variables or command-line arguments — used for default values you expect users to customize.

---

### 🔴 Advanced / Scenario-Based

---

**Q12. You have 200 servers and need to run a playbook only on servers that have less than 20% disk space. How do you do it?**

> Use Ansible's `when` conditional combined with gathered facts or a custom fact. First gather disk usage facts, then apply a condition to filter which hosts execute the task. Alternatively, use a dynamic inventory script that queries monitoring tools to return only the affected hosts.

---

**Q13. What is Ansible Galaxy and when would you use it?**

> Ansible Galaxy is a community hub for sharing and downloading pre-built roles. Use it to: initialize a new role structure (`ansible-galaxy role init <name>`), download community roles instead of writing from scratch (e.g., a tested nginx role), and share your own roles with the community. Always review community roles before using in production.

---

**Q14. How would you configure Ansible to manage a Kubernetes cluster setup?**

> For complex setups like Kubernetes (50+ tasks, certificates, secrets, multi-node config), use Ansible Roles to separate concerns: one role for common setup, one for master nodes, one for worker nodes. Use variables files for cluster-specific config, templates for kubeconfig generation, and handlers for service restarts. Terraform creates the EC2 instances; Ansible configures them.

---

## 2. Infrastructure as Code (IaC) Interview Questions

### 🟢 Beginner

---

**Q15. What is Infrastructure as Code (IaC)?**

> IaC is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than manual processes. Instead of logging into a console and clicking to create servers, you write code (scripts/templates) that defines your infrastructure. Benefits: version control, repeatability, consistency, auditability, and faster provisioning.

---

**Q16. What are the cloud-specific IaC tools and what is their limitation?**

| Cloud | Native IaC Tool | Limitation |
|-------|----------------|------------|
| AWS | CloudFormation (CFT) | AWS only |
| Azure | Azure Resource Manager (ARM) | Azure only |
| GCP | Deployment Manager | GCP only |
| OpenStack | Heat Templates | OpenStack only |

> The limitation is that these tools are cloud-specific. If your organization migrates or uses a hybrid cloud model, you need to rewrite everything in a different tool — wasting enormous effort.

---

**Q17. What is the difference between IaC and API as Code?**

> IaC is the broad concept of writing code to manage infrastructure. Tools like CFT and ARM are IaC but cloud-specific. API as Code (Terraform's approach) takes it further — one tool talks to any cloud provider's REST API. You write one HCL script, tell Terraform your provider is AWS or Azure, and Terraform translates it into the correct API calls. Migration between clouds becomes a config change, not a rewrite.

---

**Q18. What is a hybrid cloud model and how does it create IaC challenges?**

> Hybrid cloud is when an organization uses multiple cloud providers simultaneously — e.g., AWS for storage and Azure for DevOps services. The IaC challenge is that you now need to learn and maintain multiple provider-specific tools (CFT for AWS, ARM for Azure). Terraform solves this by being a single tool that handles all providers from one codebase.

---

**Q19. What is an API and why is it important in the context of DevOps?**

> An API (Application Programming Interface) is a programmatic interface that lets you interact with an application without a UI. Instead of manually opening a browser and clicking through GitHub, you send an HTTP request to the GitHub API and get structured data back. In DevOps, APIs enable automation — Terraform uses cloud APIs to create resources, webhooks use APIs to trigger pipelines, and monitoring tools expose APIs for alerts.

---

### 🟡 Intermediate

---

**Q20. Why would you choose Terraform over AWS CloudFormation?**

> CFT is AWS-only. Key reasons to prefer Terraform:
> - Works with 100+ providers (AWS, Azure, GCP, Kubernetes, Datadog, etc.)
> - Single language (HCL) instead of learning multiple tools
> - Smooth migration between clouds — change provider block, update resource names
> - Larger open-source community and module ecosystem
> - Better state management and collaboration features
> - Cleaner, more readable syntax than CFT's JSON/YAML

---

## 3. Terraform Interview Questions

### 🟢 Beginner

---

**Q21. What is Terraform and what problem does it solve?**

> Terraform is an open-source Infrastructure as Code tool by HashiCorp. It solves the problem of managing infrastructure across multiple cloud providers with a single tool. Instead of learning AWS CFT, Azure ARM, and GCP Deployment Manager, you write HCL (HashiCorp Configuration Language) and Terraform converts it into the correct API calls for whichever provider you specify.

---

**Q22. What are the four main Terraform commands?**

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize provider plugins and backend |
| `terraform plan` | Dry run — shows what will change without executing |
| `terraform apply` | Execute the changes on the cloud provider |
| `terraform destroy` | Delete all resources defined in the config |

---

**Q23. What is a Terraform provider and how do you configure it?**

> A provider is a plugin that allows Terraform to interact with a specific cloud or service API. You declare it in your config and Terraform downloads it during `init`.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
  required_version = ">= 1.2.0"
}

provider "aws" {
  region = "us-west-2"
}
```

---

**Q24. What is a Terraform State file and why is it critical?**

> The state file (`terraform.tfstate`) is Terraform's memory — it tracks every resource Terraform has created, including IDs, configuration, and metadata. Terraform uses it to:
> - Know what already exists (avoid re-creating resources)
> - Calculate what changes need to be made
> - Map config to real infrastructure
>
> It is critical because **without it, Terraform cannot manage your infrastructure**. If the state file is lost or corrupted, Terraform loses track of everything it created.

---

**Q25. What is `terraform plan` and why should you always run it before `apply`?**

> `terraform plan` is a dry run that shows exactly what Terraform will create, modify, or destroy — without making any actual changes. Always run it first to:
> - Catch mistakes before they affect real infrastructure
> - Review changes for unintended side effects
> - Get team approval (show the plan output in PRs)
> - Understand cost implications of resource changes

---

### 🟡 Intermediate

---

**Q26. What are the problems with storing the Terraform state file locally?**

> Three critical problems:
> 1. **Security risk** — State files contain sensitive data (IPs, credentials, KMS keys). Locally it's unencrypted and accessible to anyone with filesystem access
> 2. **Team collaboration failure** — If two engineers clone the repo and run `terraform apply`, they each get separate state files. These diverge and can never be reliably merged
> 3. **No locking** — Parallel execution by multiple users can corrupt the state, leading to inconsistent infrastructure

---

**Q27. What is a Terraform remote backend? What is the recommended setup on AWS?**

> A remote backend stores the state file in a centralized, secure, shared location instead of locally. The recommended AWS setup:

```
Remote Backend = S3 bucket (stores state file)
           +
State Locking = DynamoDB table (prevents parallel execution)
```

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

> S3 stores the state securely with versioning. DynamoDB locks the state so only one `apply` can run at a time.

---

**Q28. Why is DynamoDB used alongside S3 for Terraform state?**

> DynamoDB provides **state locking**. Without it, if two engineers or two pipeline runs execute `terraform apply` simultaneously, both read the same state, make conflicting changes, and write back corrupted state — potentially destroying infrastructure. DynamoDB locks the state file when one apply starts and releases it when done. Anyone else trying to apply simultaneously gets a lock error and must wait.

---

**Q29. What are Terraform modules and when should you use them?**

> Modules are reusable, self-contained packages of Terraform configuration. Use them when:
> - The same infrastructure pattern is needed across multiple environments (dev/staging/prod)
> - Multiple teams need the same base setup (e.g., every team needs S3 + DynamoDB for their state backend)
> - You want to enforce organizational standards
>
> Instead of copy-pasting 50 lines of config in 10 places, write a module once and reference it everywhere. One fix propagates to all consumers.

---

**Q30. What are `variables.tf` / `input.tf` and `outputs.tf` and why should they be separate files?**

> `variables.tf` declares input variables — the parameters your Terraform config accepts. `outputs.tf` declares what information Terraform should print after resources are created (e.g., EC2 instance public IP, RDS endpoint).
>
> Keeping them separate means: config changes happen in one predictable file, code reviews are cleaner (variable changes are low-risk vs. resource changes), and consumers of your modules know exactly what inputs are needed and what outputs to expect.

```hcl
# variables.tf
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}

# outputs.tf
output "instance_public_ip" {
  value = aws_instance.app.public_ip
}
```

---

**Q31. What is `terraform init` doing internally?**

> `terraform init` reads your `required_providers` block and downloads the corresponding provider plugins from the Terraform registry into a `.terraform` folder. It also initializes the backend configuration (local or remote). You must re-run `terraform init` whenever you:
> - Add a new provider
> - Change the backend configuration
> - Upgrade provider versions

---

### 🔴 Advanced / Scenario-Based

---

**Q32. A team member accidentally deleted the `terraform.tfstate` file. What do you do?**

> **If using remote backend (S3 with versioning):** Restore the previous version from S3 — no data loss.
>
> **If local state:** Disaster recovery via `terraform import`:
> ```bash
> terraform import aws_instance.app i-0abc123def456
> # Repeat for every resource — time-consuming
> ```
>
> **Prevention:**
> - Always use remote backends (never local state in teams)
> - Enable S3 versioning on the state bucket
> - Set bucket policy to deny `DeleteObject` except for authorized roles
> - This is exactly why local state is an anti-pattern for teams

---

**Q33. What are the major problems/limitations of Terraform?**

> 1. **State file is a single point of failure** — corruption or loss means you lose track of your entire infrastructure
> 2. **Manual cloud changes break Terraform** — if someone modifies a resource directly in the AWS console, Terraform's state is out of sync. You must run `terraform refresh` to reconcile
> 3. **Not GitOps-friendly** — Terraform doesn't auto-detect drift between your Git config and actual cloud state; it needs to be explicitly triggered
> 4. **Complex at scale** — managing hundreds of accounts/environments with Terraform requires significant tooling (Terragrunt, Atlantis) to remain manageable
> 5. **Not a configuration management tool** — Terraform creates infrastructure; it doesn't configure software on servers. Using it for that crosses into Ansible/Chef territory and creates messy, hard-to-maintain code

---

**Q34. Your organization uses AWS today but is planning to migrate to Azure in 6 months. How does Terraform help?**

> With Terraform, migration is a structured update rather than a rewrite:
> 1. Update `provider` block from `hashicorp/aws` to `hashicorp/azurerm`
> 2. Map AWS resource types to Azure equivalents (e.g., `aws_instance` → `azurerm_linux_virtual_machine`, S3 → Azure Blob Storage)
> 3. Update variable values (region names, image IDs, etc.)
> 4. The overall file structure, module logic, variable files, and output files remain identical
>
> Without Terraform, you'd be rewriting hundreds of CFT scripts from scratch in ARM template syntax — a completely different language and paradigm.

---

**Q35. What is the ideal Terraform setup for a production organization?**

> ```
> Git Repository (Terraform .tf files — version controlled, PR-reviewed)
>         ↓
> CI/CD Pipeline (Jenkins / GitHub Actions — runs plan on PR, apply on merge)
>         ↓
> Terraform Execution
>         ↓ (reads/writes state)
> S3 Bucket (remote state) + DynamoDB (state locking)
>         ↓ (API calls)
> Cloud Provider (AWS / Azure / GCP)
>         ↓ (resource output)
> outputs.tf (returns IPs, endpoints, names to pipeline logs)
> ```
>
> Key rules: Never store state in Git. Never run `apply` manually from a local machine in prod. Always enforce peer review of `terraform plan` output before merge. Isolate state files per environment (dev/staging/prod separate state files to minimize blast radius).

---

## 4. Jenkins & CI/CD Interview Questions

### 🟢 Beginner

---

**Q36. What is CI/CD? Explain the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment.**

> **Continuous Integration (CI):** Every code commit automatically triggers a build and test pipeline. Goal: catch integration bugs early.
>
> **Continuous Delivery:** Extends CI — the artifact is automatically built, tested, and prepared for release. A human approves before it goes to production.
>
> **Continuous Deployment:** Fully automated — every passing commit goes to production with no human approval gate.
>
> Most enterprises practice Continuous Delivery (not full Deployment) because production deployments require human sign-off for compliance and risk management.

---

**Q37. What is a Jenkinsfile and why should it be stored in source control?**

> A Jenkinsfile is a text file that defines a Jenkins pipeline as code using Groovy DSL. Storing it in source control means:
> - Pipeline changes are versioned and auditable
> - Changes go through code review (PR process)
> - Pipeline is consistent across all branches
> - If Jenkins is rebuilt from scratch, all pipelines are automatically recovered
> - Different branches can have different pipeline behaviors

---

**Q38. What is the difference between Declarative and Scripted Jenkins pipeline?**

> **Declarative:** Uses `pipeline {}` block — structured, enforced syntax, easier to read. Recommended for 95% of use cases.
> **Scripted:** Uses `node {}` block — full Groovy code, more flexible but harder to maintain and read.

```groovy
// Declarative (recommended):
pipeline {
  agent any
  stages {
    stage('Build') {
      steps { sh 'mvn package' }
    }
  }
}

// Scripted:
node {
  stage('Build') {
    sh 'mvn package'
  }
}
```

---

**Q39. What are the different ways to trigger a Jenkins pipeline?**

| Method | How | Problem |
|--------|-----|---------|
| Poll SCM | Jenkins checks Git on a schedule | Costly, delay |
| Build Triggers | Time-based cron job | Same delay problem |
| **Webhooks** ✅ | GitHub notifies Jenkins instantly | Best approach |

> Webhooks are the best approach. GitHub sends a JSON payload to Jenkins the moment a developer pushes code — instant trigger, no polling overhead, no delay.

---

**Q40. What is the purpose of the `input` step in Jenkins?**

> The `input` step pauses the pipeline and waits for human approval before continuing. Used to implement approval gates before production deployments.

```groovy
stage('Deploy to Production') {
  steps {
    input message: 'Approve deployment to production?',
          submitter: 'release-manager,senior-devops'
    sh './deploy-prod.sh'
  }
}
```

---

### 🟡 Intermediate

---

**Q41. How do you handle secrets in Jenkins? What is wrong with using environment variables directly?**

> Hardcoding secrets as `env.PASSWORD = "abc123"` exposes them in: Git history, build logs, and to anyone with repository read access. Correct approaches:
> 1. **HashiCorp Vault** (enterprise standard) — secrets stored externally, fetched at runtime
> 2. **Jenkins Credentials Plugin** — secrets stored in Jenkins credential store, masked in logs
> 3. **AWS Secrets Manager** — if running on AWS
>
> Always use `withCredentials()` block so credentials are masked in console output.

---

**Q42. What are Jenkins Shared Libraries and when would you use them?**

> Shared libraries are reusable Groovy code stored in a separate Git repo, callable from any Jenkinsfile across the organization. Use when:
> - Multiple teams repeat the same pipeline stages
> - You want one place to update a standard build process
> - You need to enforce org-wide pipeline standards
>
> Without shared libraries, updating the build step requires modifying 50 Jenkinsfiles. With a shared library, one update propagates everywhere.

---

**Q43. How do you build applications with multiple programming languages in one Jenkins pipeline?**

> Use Docker agents per stage. Each stage spins up a container with the right environment, executes, and is destroyed — no dependency conflicts, no manual setup on worker nodes.

```groovy
pipeline {
  agent none
  stages {
    stage('Frontend') {
      agent { docker { image 'node:18' } }
      steps { sh 'npm run build' }
    }
    stage('Backend') {
      agent { docker { image 'maven:3.8-jdk-11' } }
      steps { sh 'mvn clean package' }
    }
    stage('ML Service') {
      agent { docker { image 'python:3.11' } }
      steps { sh 'pip install -r requirements.txt' }
    }
  }
}
```

---

**Q44. What is JNLP in Jenkins?**

> JNLP (Java Network Launch Protocol) is the mechanism by which Jenkins worker/agent nodes connect to the Jenkins master. Instead of the master SSHing into the worker, the **worker initiates** the connection to the master by downloading and running a JNLP jar. Used when worker nodes are behind a firewall and the master cannot reach them directly via SSH.

---

**Q45. How do you back up Jenkins?**

> Primary backup: the `.jenkins` hidden folder in the Jenkins home directory — contains all job configs, build logs, plugin data, credentials, and node configs.
>
> ```bash
> rsync -av ~/.jenkins/ /backup/jenkins/
> ```
>
> Additional: backup external databases if used, maintain an approved plugins list, and store all Jenkinsfiles in Git (most important — pipeline-as-code means pipelines survive a Jenkins wipe).

---

### 🔴 Advanced / Scenario-Based

---

**Q46. A developer pushes code but the Jenkins pipeline doesn't trigger. Walk through your debugging steps.**

> 1. Check if webhook is configured in GitHub (Settings → Webhooks) — verify payload URL points to correct Jenkins URL
> 2. Check webhook delivery history in GitHub — did the payload send? What was the HTTP response code?
> 3. Check Jenkins logs for incoming webhook requests
> 4. Verify the Jenkins GitHub plugin is installed and configured
> 5. Check if the pipeline branch filter matches the branch that was pushed to
> 6. If using Poll SCM — verify the cron expression is correct
> 7. Check if the Jenkins job is disabled or paused

---

**Q47. Jenkins master goes down during a production deployment. What do you do and how do you prevent it?**

> **Immediate response:**
> - Restart Jenkins service (`systemctl restart jenkins`)
> - If EC2 — restart the instance or restore from snapshot
> - Re-trigger the failed pipeline from last known good state
>
> **Prevention:**
> - Store all Jenkinsfiles in Git (pipelines survive a Jenkins wipe)
> - Regular `.jenkins` backups with automated restore scripts
> - HA setup: active-passive Jenkins masters with shared EBS storage
> - Consider cloud-native CI (GitHub Actions) for true HA with zero maintenance

---

**Q48. How do you ensure consistent pipelines across 50+ teams in a large organization?**

> Multi-layer strategy:
> 1. **Shared Libraries** — all teams use standard `buildJava()`, `deployToK8s()` functions
> 2. **Pipeline Templates** — teams inherit a base pipeline, only customize approved stages
> 3. **Jenkins Configuration as Code (JCasC)** — Jenkins config stored in Git YAML, not clicked through UI
> 4. **Managed plugin list** — teams cannot install ad-hoc plugins; central team maintains approved versions
> 5. **RBAC** — teams trigger and monitor their pipelines but cannot modify global config
> 6. **Audit logging** — every pipeline change tracked

---

## 5. Mixed / Cross-Topic Questions

---

**Q49. What is the difference between Ansible and Terraform? When do you use each?**

| | Terraform | Ansible |
|-|-----------|---------|
| **Primary use** | Infrastructure provisioning | Configuration management |
| **Examples** | Create EC2, S3, VPC, RDS | Install nginx, configure app, deploy code |
| **State** | Stateful (tracks resources) | Stateless (idempotent tasks) |
| **Language** | HCL | YAML |
| **When to use** | Creating cloud resources | Configuring servers after creation |

> They are **complementary**, not competing. Standard workflow: Terraform creates 3 EC2 instances → Ansible configures them as Kubernetes master and worker nodes.

---

**Q50. How does a complete DevOps pipeline look end-to-end?**

```
Developer commits code to GitHub
        ↓ (webhook)
Jenkins pipeline triggered
        ↓
Stage 1: Checkout code (Git plugin)
        ↓
Stage 2: Build (Maven / npm / pip)
        ↓
Stage 3: Unit Tests
        ↓
Stage 4: Code Quality (SonarQube)
        ↓
Stage 5: Security Scan (AppScan / Trivy)
        ↓
Stage 6: Docker build & push to registry
        ↓
Stage 7: Update K8s manifest in Git (new image tag)
        ↓
Argo CD detects change (GitOps)
        ↓
Deploy to Kubernetes cluster
        ↓
Monitoring (Prometheus + Grafana)
```

> Infrastructure behind all this is created by **Terraform**. Servers are configured by **Ansible**. The application is deployed by **Jenkins + Argo CD**.

---

**Q51. What is GitOps and how does it differ from traditional CI/CD?**

> GitOps is a deployment model where **Git is the single source of truth** for infrastructure and application state. Any change to production must go through a Git commit — no manual console changes, no ad-hoc scripts.
>
> Traditional CI/CD: pipeline **pushes** changes to the environment.
> GitOps: a controller (Argo CD, Flux) **pulls** from Git and reconciles the environment to match Git state.
>
> Benefit: automatic drift detection and correction. If someone manually changes something in production, GitOps will revert it to match Git.

---

## 6. Scenario-Based Questions (Senior Level)

---

**Q52. Your production deployment failed halfway through. Terraform applied some resources but not others. How do you recover?**

> Terraform is transactional at the resource level, not the overall plan. When a partial apply happens:
> 1. Run `terraform plan` to see current state vs desired state
> 2. Terraform will show which resources were created and which failed
> 3. Fix the root cause of the failure (e.g., IAM permissions, resource limits)
> 4. Re-run `terraform apply` — Terraform is idempotent, it will only create what's missing
> 5. Never manually delete partially created resources before running `terraform apply` — let Terraform reconcile

---

**Q53. A security audit reveals hardcoded AWS credentials in a Jenkinsfile committed to a public GitHub repo. What do you do?**

> **Immediate (treat as a security incident):**
> 1. **Rotate the credentials immediately** in AWS IAM — assume they're already compromised
> 2. Check AWS CloudTrail for any unauthorized API calls using those credentials
> 3. Remove credentials from the file and rewrite Git history: `git filter-branch` or BFG Repo Cleaner
> 4. Force-push cleaned history to all branches
>
> **Fix the root cause:**
> - Store credentials in Jenkins Credentials store or HashiCorp Vault
> - Use `withCredentials()` block in Jenkinsfile
> - Add pre-commit hooks with secret scanning (GitGuardian, `detect-secrets`)
> - Enable GitHub secret scanning to auto-detect future leaks

---

**Q54. Your Ansible playbook runs successfully but the nginx service keeps failing on 3 out of 100 servers. How do you debug?**

> 1. Re-run only on failed hosts: `ansible -i inventory failed_hosts -m service -a "name=nginx state=status" -vvv`
> 2. Check if the 3 failing servers have different OS versions (`gather_facts` → `ansible_distribution_version`)
> 3. Check if nginx was already running with a conflicting config before Ansible ran
> 4. Check port conflicts — something else may be using port 80/443
> 5. Use `--step` flag to run playbook interactively task by task on a failing host
> 6. Use `-vvv` verbose flag for full SSH connection and task execution details
> 7. Add error handling with `block/rescue` to capture and log failures

---

**Q55. Your organization wants to adopt IaC from scratch. How would you design the entire setup?**

> **Phase 1 — Foundation:**
> - Choose Terraform as the IaC tool
> - Set up remote backend: S3 + DynamoDB for state management
> - Create a Git repository for all Terraform code with branch protection
>
> **Phase 2 — Structure:**
> - Separate state files per environment (dev/staging/prod)
> - Create shared modules for common patterns (EC2, RDS, VPC)
> - Enforce `variables.tf` and `outputs.tf` in all modules
>
> **Phase 3 — Automation:**
> - Integrate with Jenkins/GitHub Actions: `plan` on PR, `apply` on merge to main
> - Require peer review of `terraform plan` output before merge
> - Never allow manual `apply` from local machines in prod
>
> **Phase 4 — Configuration:**
> - Use Ansible for all post-provisioning server configuration
> - Store Ansible roles in a separate Git repo
> - Use Ansible Vault or HashiCorp Vault for secrets
>
> **Phase 5 — Governance:**
> - Tag all resources via Terraform for cost tracking
> - Set up OPA/Sentinel policies to enforce security standards in Terraform
> - Alerting on manual console changes via AWS Config

---

## 📋 Quick Reference — Key Differences

| Topic | Option A | Option B | Winner |
|-------|----------|----------|--------|
| IaC tool | CFT / ARM (cloud-specific) | Terraform (provider-agnostic) | Terraform |
| Trigger Jenkins | Poll SCM | Webhooks | Webhooks |
| Secrets | Env variables | HashiCorp Vault | Vault |
| Terraform state | Local / Git | S3 + DynamoDB | S3 + DynamoDB |
| Complex playbooks | Single YAML file | Ansible Roles | Roles |
| Multi-lang builds | Install deps on agent | Docker agents | Docker agents |
| Server config | Manual / scripts | Ansible | Ansible |
| Infra provisioning | Manual console | Terraform | Terraform |

---

## ✅ Master Checklist — Before Your Interview

### Ansible
- [ ] Explain passwordless SSH setup from memory
- [ ] Difference: ad-hoc vs playbook (with examples)
- [ ] Ansible Role folder structure and purpose of each folder
- [ ] What handlers do and when they're triggered
- [ ] Difference: `vars/` vs `defaults/`

### Terraform
- [ ] Four Terraform commands and what each does
- [ ] Why local state is dangerous (3 reasons)
- [ ] S3 + DynamoDB remote backend setup
- [ ] What modules are and when to use them
- [ ] Three disadvantages of Terraform
- [ ] Ideal Terraform org setup (the full diagram)

### Jenkins / CI-CD
- [ ] Explain your CI/CD pipeline end-to-end with tool names
- [ ] Difference: CI vs Continuous Delivery vs Continuous Deployment
- [ ] Why webhooks beat Poll SCM
- [ ] How to handle secrets properly
- [ ] What shared libraries are and why they matter
- [ ] Docker agents for multi-language builds
- [ ] How to add approval gates (`input` step)

### Cross-Topic
- [ ] Terraform vs Ansible — when to use each
- [ ] What GitOps is and how it differs from push-based CI/CD
- [ ] Full end-to-end DevOps pipeline explanation
