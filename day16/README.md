# Infrastructure as Code (IaC) & Terraform - Day 16 | Complete DevOps Course
> 📺 [Watch the Video](https://www.youtube.com/watch?v=Z6T2r3Xhk5k&list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&index=18) | By Abhishek

---

## 📌 What This Video Covers

- What is Infrastructure as Code (IaC)?
- The Problem with Cloud-Specific Tools
- What is Terraform and Why It Exists
- What is an API and API as Code
- How Terraform Works Internally

---

## 1. 🧩 The Problem — Why IaC Tools Were Not Enough

As a DevOps Engineer, you automate infrastructure using cloud-specific tools:

| Cloud Provider | Native IaC Tool |
|----------------|-----------------|
| AWS | CloudFormation Templates (CFT) |
| Azure | Azure Resource Manager (ARM) |
| GCP | Deployment Manager |
| OpenStack | Heat Templates |

### The Pain Point:
- You write **hundreds of scripts** for AWS using CFT ✅
- Company migrates to **Azure** → All CFT scripts are **useless** ❌
- You rewrite everything in ARM templates ✅
- Company moves to **On-Premise (OpenStack)** → ARM scripts are **useless** ❌
- You rewrite everything in Heat templates... and so on 😩

### Hybrid Cloud Makes it Worse:
Organizations often use **multiple cloud providers simultaneously**:
- AWS for storage (S3, RDS)
- Azure for DevOps services
- On-premise for sensitive workloads

As a DevOps engineer you now need to **learn and maintain multiple tools** — that's unsustainable.

---

## 2. ✅ The Solution — Terraform

**Terraform** is an open-source IaC tool by **HashiCorp**.

> **Core Idea:** Write once, deploy anywhere. Instead of learning AWS CFT + Azure ARM + Heat Templates, just learn **one tool — Terraform**.

```
You write Terraform scripts
        ↓
Terraform reads your provider (AWS / Azure / GCP)
        ↓
Terraform converts your script → API calls to that provider
        ↓
Cloud provider executes the action and returns result
```

### Key Benefits:
- **Provider-agnostic** — works with 100+ providers
- **Smooth migration** — moving from AWS to Azure = updating provider config + some module tweaks
- **One language** — HCL (HashiCorp Configuration Language) for everything
- **Community modules** — pre-built modules for almost every resource

---

## 3. 🔌 What is an API?

> API = Application Programming Interface

Two ways to interact with any application (e.g., GitHub):

| Method | How |
|--------|-----|
| **Manual (UI)** | Open browser → login → click through interface |
| **Programmatic (API)** | Run a script → HTTP GET/POST request → get result back |

### Example:
```bash
# Manually: open browser, go to github.com, find repo info
# Programmatically via API:
curl https://api.github.com/repos/abhishekf5/ansible-examples
```

APIs allow **automation without human interaction** — the foundation of DevOps.

---

## 4. ⚙️ API as Code — How Terraform Works

Terraform takes the concept of IaC further with **API as Code**:

- Every cloud provider exposes **REST APIs**
- Terraform has **provider plugins** that wrap those APIs
- You write simple **HCL config** → Terraform translates it to the right API call

```hcl
# You write this (provider.tf):
provider "aws" {
  region = "us-east-1"
}

# Terraform internally converts to AWS API call:
# POST https://ec2.amazonaws.com/ → CreateInstance(...)
```

### IaC vs API as Code:

| Concept | Tools | Limitation |
|---------|-------|------------|
| **Infrastructure as Code** | CFT, ARM, Heat | Cloud-specific, not portable |
| **API as Code (Terraform)** | Terraform | One tool, any provider |

---

## 5. 🏗️ Real-World Architecture Example

**Flipkart-style hybrid setup using Terraform:**

```
Terraform Scripts
      ├── provider "aws"   → Creates S3, RDS
      ├── provider "azurerm" → Creates Azure DevOps pipelines
      └── provider "openstack" → Creates on-premise VMs
```

All managed from a **single codebase**.

---

## 6. 🔄 Terraform vs Ansible — When to Use What

| | Terraform | Ansible |
|-|-----------|---------|
| **Purpose** | Create infrastructure | Configure infrastructure |
| **Examples** | Create EC2, S3, VPC | Install nginx, deploy app |
| **State** | Stateful (tracks resources) | Stateless |
| **Language** | HCL | YAML |
| **Use together?** | ✅ Yes — Terraform creates, Ansible configures |

---

## 7. 🎯 Interview Questions

> *Questions curated from the perspective of a 10-year hiring manager. These reflect what actually gets asked at mid to senior DevOps roles.*

---

### 🟢 Beginner Level

**Q1. What is Infrastructure as Code? Why is it better than manual provisioning?**

> IaC means managing and provisioning infrastructure through machine-readable config files rather than manual processes. Benefits: version control, repeatability, consistency across environments, faster provisioning, and reduced human error.

---

**Q2. What is the difference between IaC and API as Code?**

> IaC is the broad concept — writing scripts to automate infrastructure. Tools like CFT and ARM are IaC but cloud-specific. API as Code (what Terraform does) is an evolution where a single tool talks to any provider's API, making scripts portable and provider-agnostic.

---

**Q3. Why would you choose Terraform over AWS CloudFormation?**

> CFT is AWS-only. If you work with multiple cloud providers or need to migrate in the future, Terraform's provider-agnostic approach saves massive effort. Terraform also has a larger community, better module ecosystem, and a more readable language (HCL vs JSON/YAML in CFT).

---

**Q4. What is HCL?**

> HashiCorp Configuration Language — the declarative language used to write Terraform scripts. It is human-readable and designed to describe infrastructure resources concisely.

---

### 🟡 Intermediate Level

**Q5. What is a Terraform provider? Give examples.**

> A provider is a plugin that lets Terraform interact with a specific cloud or service API. Examples: `hashicorp/aws`, `hashicorp/azurerm`, `hashicorp/google`, `hashicorp/kubernetes`. You declare it in `provider.tf` and Terraform downloads it automatically.

---

**Q6. Explain Terraform state. Why is it important?**

> Terraform maintains a state file (`terraform.tfstate`) that maps your config to real-world resources. It tracks what exists so Terraform knows what to create, update, or destroy. Without state, Terraform can't know the current status of your infrastructure.
>
> **Follow-up trap:** *Where do you store state in a team environment?*
> → Remote backends like S3 + DynamoDB (for locking), Azure Blob Storage, or Terraform Cloud — never local in a team.

---

**Q7. What are Terraform modules? Why use them?**

> Modules are reusable, self-contained packages of Terraform configurations. Instead of repeating the same EC2 setup in 10 places, you write it once as a module and call it wherever needed. Promotes DRY (Don't Repeat Yourself) and maintainability.

---

**Q8. What is the difference between `terraform plan` and `terraform apply`?**

> `terraform plan` is a dry run — shows what changes will be made without executing them. `terraform apply` actually makes those changes. Best practice: always review `plan` output before `apply`, especially in production.

---

**Q9. What happens if two engineers run `terraform apply` at the same time?**

> Race condition — they could corrupt the state file. This is solved using **state locking** with a backend like DynamoDB (on AWS). When one apply is running, the state is locked and the second engineer gets a lock error until the first completes.

---

**Q10. How is Terraform different from Ansible in a real project?**

> They complement each other. Terraform is used to **provision** infrastructure (create VMs, networks, databases). Ansible is used to **configure** that infrastructure (install software, manage files, configure services). A typical pipeline: Terraform creates 3 EC2 instances → Ansible configures them as Kubernetes master and worker nodes.

---

### 🔴 Advanced / Scenario-Based

**Q11. Your company is migrating from AWS to Azure. How would Terraform help vs using CloudFormation?**

> With CFT, you'd rewrite hundreds of scripts from scratch. With Terraform, you update the `provider` block from `aws` to `azurerm`, update resource names to Azure equivalents (e.g., `aws_instance` → `azurerm_virtual_machine`), and adjust resource-specific arguments. The structure, logic, variables, and modules largely stay the same — significantly reducing migration effort.

---

**Q12. What is `terraform destroy`? When would you use it and what precautions would you take?**

> `terraform destroy` tears down all resources defined in your config. Use cases: cleaning up dev/test environments, cost saving. Precautions: always run `plan -destroy` first, ensure you're in the right workspace/environment, protect production state files, and consider `prevent_destroy` lifecycle rules on critical resources.

---

**Q13. A junior engineer accidentally deleted the `terraform.tfstate` file. What do you do?**

> This is a disaster scenario. Options:
> 1. If using remote backend (S3) — restore from the backend; it's safe.
> 2. If local state — use `terraform import` to re-import existing resources back into a new state file one by one.
> 3. As prevention — always use remote backends with versioning enabled (S3 versioning) so you can restore previous state.
>
> This is why storing state locally in production is a critical anti-pattern.

---

**Q14. What is the difference between `terraform.tfvars` and `variables.tf`?**

> `variables.tf` **declares** variables — name, type, description, and optional default. `terraform.tfvars` **assigns values** to those variables for a specific environment. This separation lets you reuse the same module across dev/staging/prod by just swapping the `.tfvars` file.

---

**Q15. How do you handle secrets in Terraform (e.g., DB passwords, API keys)?**

> Never hardcode secrets in `.tf` files or commit them to Git. Best practices:
> - Use environment variables (`TF_VAR_db_password`)
> - Use a secrets manager (AWS Secrets Manager, HashiCorp Vault)
> - Mark variables as `sensitive = true` to prevent them from appearing in logs
> - Use remote backends that encrypt state at rest (state files can contain secrets)

---

## 8. 🚀 What's Coming Next (Day 17)

- Terraform installation
- Writing first Terraform script
- Creating EC2 instances on AWS using Terraform
- Live project walkthrough

---

## 📚 Resources

- 🔗 [Terraform Official Docs](https://developer.hashicorp.com/terraform/docs)
- 🔗 [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- 🔗 [HashiCorp Learn](https://developer.hashicorp.com/terraform/tutorials)

---

## ✅ Revision Checklist

- [ ] Can explain IaC and why it matters
- [ ] Can explain the problem with cloud-specific tools (CFT, ARM, Heat)
- [ ] Can explain what Terraform is and how it solves the problem
- [ ] Understand what an API is and why it matters for DevOps
- [ ] Can explain API as Code vs Infrastructure as Code
- [ ] Know when to use Terraform vs Ansible
- [ ] Can answer all 15 interview questions above confidently
