# Jenkins & CI/CD Interview Questions - Day 21 | Complete DevOps Course
> 📺 [Watch the Video](https://www.youtube.com/watch?v=Z6T2r3Xhk5k&list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&index=18) | By Abhishek

---

## 📌 What This Video Covers

- CI/CD Pipeline explanation (org-level answer)
- Ways to trigger Jenkins pipelines
- Jenkins backup strategies
- Handling Secrets in Jenkins
- Shared Libraries/Modules
- Multi-language builds with Docker agents
- Auto-scaling Jenkins worker nodes
- Adding worker nodes, installing plugins
- JNLP and common Jenkins plugins

---

## 1. 🔄 CI/CD Pipeline — How to Answer in Interviews

> Interviewers want to hear YOUR org's pipeline, not a textbook definition. Structure your answer like this:

### Standard Java Pipeline Answer:
```
Developer commits code to GitHub
        ↓
Jenkins pipeline auto-triggered (via Webhook)
        ↓
Stage 1: Checkout — Pull code from GitHub
        ↓
Stage 2: Build — Maven compiles the code
        ↓
Stage 3: Code Quality — SonarQube static analysis
        ↓
Stage 4: Security Scan — AppScan (SAST/DAST)
        ↓
Stage 5: Build Docker Image & push to registry
        ↓
Stage 6: Update K8s manifest in Git (new image tag)
        ↓
Argo CD detects change → deploys to Kubernetes cluster
```

### If Not Using Kubernetes:
Replace the last two steps with:
> "We deploy the application artifact onto EC2 instances using shell scripts / Ansible"

### Sample Jenkinsfile Structure:
```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { git 'https://github.com/org/repo.git' }
    }
    stage('Build') {
      steps { sh 'mvn clean package' }
    }
    stage('Code Quality') {
      steps { sh 'mvn sonar:sonar' }
    }
    stage('Docker Build & Push') {
      steps { sh 'docker build -t myapp:${BUILD_NUMBER} .' }
    }
    stage('Deploy') {
      steps { sh './scripts/update-manifest.sh ${BUILD_NUMBER}' }
    }
  }
}
```

---

## 2. ⚡ Ways to Trigger Jenkins Pipelines

| Method | How It Works | Drawback |
|--------|-------------|----------|
| **Poll SCM** | Jenkins checks GitHub on a schedule (cron) | Costly, delay between commit and trigger |
| **Build Triggers** | Time-based scheduled execution | Same delay problem |
| **Webhooks** ✅ | GitHub notifies Jenkins instantly on any event | Best approach |

### How Webhooks Work:
```
Developer pushes code to GitHub
        ↓
GitHub sends JSON payload to Jenkins URL
(Contains: committer, PR number, branch, etc.)
        ↓
Jenkins receives API call → triggers pipeline immediately
```

### Configure in GitHub:
> Settings → Webhooks → Add webhook → Payload URL: `http://<jenkins-url>/github-webhook/`

---

## 3. 💾 Jenkins Backup

```bash
# Primary backup — the .jenkins folder (hidden folder)
rsync -av ~/.jenkins/ /backup/jenkins/

# What's inside .jenkins:
# - Pipeline job configs
# - Build logs
# - Plugin data
# - Credentials
# - Node configs
```

### Backup Strategy:
- Use `rsync` to sync `.jenkins` to EBS volume or S3
- If using external DB (large orgs) — backup the DB separately
- Take EBS snapshots for full system backup
- Backup plugins list separately for disaster recovery

---

## 4. 🔐 Handling Secrets in Jenkins

> Never hardcode secrets in Jenkinsfile or store as plain text.

### Best Practice — HashiCorp Vault Integration:
```groovy
pipeline {
  agent any
  stages {
    stage('Deploy') {
      steps {
        withVault(configuration: [vaultUrl: 'https://vault.company.com'],
                  vaultSecrets: [[path: 'secret/db', secretValues: [
                    [envVar: 'DB_PASSWORD', vaultKey: 'password']
                  ]]]) {
          sh 'deploy.sh'
        }
      }
    }
  }
}
```

### Options (in order of preference):
1. **HashiCorp Vault** — industry standard, integrates with Terraform/Ansible/Jenkins
2. **Jenkins Credentials Plugin** — built-in, good for small teams
3. **AWS Secrets Manager** — if fully on AWS
4. **Environment variables** — only for non-sensitive config

---

## 5. 📦 Shared Libraries / Shared Modules

Used when **multiple teams need the same pipeline logic** — write once, reuse everywhere.

```
groovy-shared-library/
├── vars/
│   ├── buildJava.groovy       ← callable as buildJava()
│   ├── deployToK8s.groovy     ← callable as deployToK8s()
│   └── sendSlackNotification.groovy
└── src/
    └── org/company/Utils.groovy
```

### In Jenkinsfile:
```groovy
@Library('my-shared-library') _

pipeline {
  agent any
  stages {
    stage('Build') { steps { buildJava() } }
    stage('Deploy') { steps { deployToK8s() } }
  }
}
```

> **Benefit:** Team A, B, C all use the same `buildJava()` — one fix propagates everywhere.

---

## 6. 🐳 Multi-Language Builds with Docker Agents

For apps with **multiple programming languages** (e.g., React frontend + Java backend + Python service):

```groovy
pipeline {
  agent none  // No global agent
  stages {
    stage('Build Frontend') {
      agent { docker { image 'node:18' } }
      steps { sh 'npm install && npm run build' }
    }
    stage('Build Backend') {
      agent { docker { image 'maven:3.8-jdk-11' } }
      steps { sh 'mvn clean package' }
    }
    stage('Build ML Service') {
      agent { docker { image 'python:3.11' } }
      steps { sh 'pip install -r requirements.txt && python build.py' }
    }
  }
}
```

> **Benefit:** No dependencies installed on worker node. Docker containers spin up, execute, and are deleted — zero maintenance overhead.

---

## 7. 📈 Auto-Scaling Jenkins Worker Nodes (AWS)

Used when teams need **extra workers during peak load** (e.g., Christmas releases):

```
Jenkins Master (EC2)
        ↓
AWS EC2 Auto Scaling Group
        ↓
Worker nodes scale in/out based on build queue length
```

### Steps:
1. Install **Amazon EC2 Plugin** in Jenkins
2. Configure AWS credentials in Jenkins
3. Set up Launch Template for worker EC2
4. Configure auto-scaling rules (min/max workers, idle timeout)
5. Jenkins automatically provisions/terminates workers based on load

---

## 8. ➕ Adding a New Worker Node

> Manage Jenkins → Manage Nodes and Clouds → New Node

```
Node Name: worker-node-01
Type: Permanent Agent

Config:
- Remote root directory: /home/jenkins
- Launch method: SSH
- Host: <worker-ip>
- Credentials: SSH private key
- Click "Launch"
```

---

## 9. 🔌 Installing Plugins

### Via UI:
> Manage Jenkins → Manage Plugins → Available → Search → Install

### Via CLI (efficient for bulk install):
```bash
java -jar jenkins-cli.jar \
  -s http://localhost:8080/ \
  install-plugin git maven-plugin docker-workflow blueocean
```

---

## 10. 🔗 JNLP — How Master-Agent Communication Works

> JNLP (Java Network Launch Protocol) is the mechanism Jenkins uses to connect master and worker nodes.

```
Jenkins Master
      ↓ (downloads jnlp jar)
Worker Node runs: java -jar agent.jar -jnlpUrl http://jenkins/...
      ↓
Worker connects to master → receives build tasks → executes → reports back
```

Used when you can't do SSH (e.g., worker behind firewall) — worker **initiates** the connection to master.

---

## 11. 🧩 Common Jenkins Plugins

| Plugin | Purpose |
|--------|---------|
| **Git Plugin** | Clone/checkout from GitHub/GitLab |
| **Maven Integration** | Build Java apps with Maven |
| **Docker Pipeline** | Use Docker agents in pipeline |
| **Kubernetes Plugin** | Dynamic agents on K8s |
| **SonarQube Scanner** | Code quality analysis |
| **Blue Ocean** | Modern pipeline UI |
| **Credentials Binding** | Inject secrets into builds |
| **Slack Notification** | Send build status to Slack |
| **Pipeline** | Core Jenkinsfile support |
| **GitHub Integration** | Webhook & PR integration |

---

## 12. 🎯 Interview Questions

> *15 questions from the lens of a hiring manager with 10 years of experience. These are the exact questions used to differentiate a junior who "used Jenkins" from a senior who "owns Jenkins".*

---

### 🟢 Beginner Level

**Q1. Explain your CI/CD pipeline end to end.**

> Don't say "it builds and deploys." Walk through every stage: checkout → build → test → code quality → security scan → Docker build → push to registry → update manifest → Argo CD deploys to K8s. Name the tools at each stage. This answer alone reveals your actual experience level.

---

**Q2. What is the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?**

> **CI:** Automatically build and test every code commit.
> **Continuous Delivery:** CI + automatically prepare the release artifact, but a human approves before production deploy.
> **Continuous Deployment:** Fully automated — code goes to production without any manual approval gate.
>
> Most orgs do Continuous Delivery (not full Deployment) because production needs human sign-off.

---

**Q3. What is a Jenkinsfile and why should it be in source control?**

> A Jenkinsfile defines your pipeline as code (Groovy DSL). Storing it in source control means: pipeline changes are versioned, auditable, reviewed via PRs, and consistent across branches. It also enables the pipeline to recover from scratch if Jenkins is rebuilt.

---

**Q4. What is the difference between Declarative and Scripted pipeline?**

> **Declarative:** Structured, uses `pipeline {}` block. Easier to read, enforces syntax, recommended for most use cases.
> **Scripted:** Groovy-based, uses `node {}` block. More flexible and powerful but harder to maintain.
> Most modern orgs use Declarative; Scripted is for edge cases needing full Groovy logic.

---

### 🟡 Intermediate Level

**Q5. How do you handle secrets in Jenkins? What's wrong with using environment variables directly?**

> Directly setting `env.PASSWORD = "abc123"` in Jenkinsfile exposes the secret in: git history, console logs, and anyone with read access to the repo. Proper approach: HashiCorp Vault for enterprise, Jenkins Credentials Plugin at minimum. Always mark credentials as secret so they're masked in logs.

---

**Q6. What are shared libraries in Jenkins? When would you use them?**

> Shared libraries are reusable Groovy code stored in a separate Git repo, callable from any Jenkinsfile. Use them when: multiple teams repeat the same pipeline stages, you want one place to update a build process, or you want to enforce org-wide standards. Without shared libraries, a change to the build process requires updating 50 Jenkinsfiles.

---

**Q7. A developer complains that the Jenkins pipeline only triggers once a day even after pushing code. What's the issue?**

> Most likely using Poll SCM or Build Triggers with a cron schedule instead of Webhooks. The fix is to configure a GitHub Webhook pointing to `http://<jenkins>/github-webhook/`. Webhooks trigger the pipeline instantly on push rather than waiting for the next scheduled poll.

---

**Q8. How would you speed up a slow Jenkins pipeline?**

> Several approaches:
> - **Parallel stages:** Run independent stages (tests, security scan) simultaneously
> - **Docker agents:** Avoid reinstalling dependencies on every run
> - **Caching:** Cache Maven/npm dependencies between builds
> - **Incremental builds:** Only build changed modules in a monorepo
> - **Distributed agents:** More worker nodes to handle parallel builds
> - **Pipeline optimization:** Identify bottleneck stage using build time metrics

---

**Q9. What is the difference between `agent any`, `agent none`, and `agent { docker {} }`?**

> `agent any` — run on any available worker node.
> `agent none` — no global agent; each stage defines its own (used for multi-language Docker builds).
> `agent { docker { image 'maven:3.8' } }` — spin up a Docker container for that stage, execute, and destroy it after.

---

**Q10. How do you back up Jenkins and restore it after a failure?**

> Back up the `.jenkins` directory using `rsync` to S3 or EBS. This contains all job configs, logs, plugins, and credentials. For large orgs using external DB, back up the DB separately. On restore: spin up new Jenkins instance, restore `.jenkins`, reinstall plugins if needed. Best practice: store `Jenkinsfile` in Git so pipelines are already version-controlled independently of Jenkins itself.

---

### 🔴 Advanced / Scenario-Based

**Q11. Your Jenkins master goes down during a critical production deployment. How do you handle this?**

> Short-term: If using Kubernetes, pod restarts automatically. If EC2, restart the instance — Jenkins resumes from last saved state. The pipeline may need to be re-triggered.
> Long-term prevention:
> - HA Jenkins setup with active-passive master
> - Store all state in external storage (not local disk)
> - All Jenkinsfiles in Git (not lost if Jenkins dies)
> - Regular `.jenkins` backups with automated restore scripts tested quarterly
> - Consider migrating to cloud-native CI (GitHub Actions, Argo Workflows) for true HA

---

**Q12. A pipeline works fine on one agent but fails on another. How do you debug it?**

> Classic agent configuration drift. Debugging steps:
> 1. Check if required tools (Java, Maven, Docker) are installed on the failing agent
> 2. Compare tool versions between agents (`java -version`, `mvn -version`)
> 3. Check environment variables and PATH on both agents
> 4. Check disk space and permissions on the failing agent
> 5. Look at workspace — stale files from previous builds?
> Long-term fix: Use Docker agents — every build gets a clean, identical container regardless of which worker runs it.

---

**Q13. How would you implement approval gates in a Jenkins pipeline before production deployment?**

> Use the `input` step in Declarative pipeline:
> ```groovy
> stage('Deploy to Production') {
>   steps {
>     input message: 'Approve production deployment?',
>           submitter: 'senior-devops,release-manager'
>     sh './deploy-prod.sh'
>   }
> }
> ```
> Jenkins pauses, sends notification (Slack/email), waits for named approvers. Build expires after timeout if no response. This implements Continuous Delivery (vs full Deployment).

---

**Q14. A security audit finds that your Jenkinsfile contains an AWS access key committed to Git. What do you do?**

> Immediate response (treat as incident):
> 1. **Rotate the key immediately** in AWS IAM — the old key is compromised
> 2. Check CloudTrail for unauthorized usage of that key
> 3. Remove the key from the Jenkinsfile and force-push to rewrite Git history (`git filter-branch` or BFG Repo Cleaner)
> 4. Add AWS credentials to Jenkins Credentials store and reference via `withCredentials()`
>
> Prevention: Add secret scanning (GitGuardian, GitHub secret scanning) to block commits containing credentials. Add a pre-commit hook locally.

---

**Q15. How do you ensure Jenkins pipelines are consistent across 50 teams in a large organization?**

> Multi-layer strategy:
> 1. **Shared Libraries** — enforce org-wide build, test, and deploy standards
> 2. **Pipeline Templates** — teams inherit from a base pipeline, can only customize approved stages
> 3. **Jenkins Configuration as Code (JCasC)** — Jenkins config stored in YAML in Git, not clicked through UI
> 4. **Plugin management** — maintain an approved plugin list; auto-install via script, don't let teams install ad-hoc
> 5. **RBAC** — teams can trigger and view their pipelines, but cannot modify global Jenkins config
> 6. **Audit logging** — track who changed what pipeline and when

---


## ✅ Revision Checklist

- [ ] Can explain your CI/CD pipeline end-to-end with tool names
- [ ] Know the difference between CI / Continuous Delivery / Continuous Deployment
- [ ] Understand Declarative vs Scripted pipeline
- [ ] Can explain Webhooks vs Poll SCM and why Webhooks win
- [ ] Know how to handle secrets (Vault / Credentials Plugin)
- [ ] Understand Shared Libraries and when to use them
- [ ] Can explain Docker agents for multi-language builds
- [ ] Know how Jenkins master-agent communication works (JNLP)
- [ ] Know at least 8 common plugins and their purpose
- [ ] Can answer all 15 interview questions above confidently
