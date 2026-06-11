# Terraform - Day 17 | Complete DevOps Course
> 📺 [Watch the Video](https://www.youtube.com/watch?v=Z6T2r3Xhk5k&list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&index=18) | By Abhishek
> 📁 [GitHub Repository](https://github.com/abhishekf5/terraform-zero-to-hero)

---

## 📌 What This File Covers

- What is Terraform and how it works (recap)
- Terraform lifecycle — init, plan, apply, destroy
- Writing your first `main.tf` file
- Terraform authentication with AWS CLI
- Local state vs Remote state
- Terraform State file — what it is and best practices
- Remote backend — S3 + DynamoDB setup
- `variables.tf` and `outputs.tf`
- Terraform Modules
- Problems / Disadvantages of Terraform
- Interview questions

---

## 1. 🔄 How Terraform Works (Quick Recap)

```
You write Terraform config (.tf files)
        ↓
Specify provider (AWS / Azure / GCP)
        ↓
Terraform converts config → API calls for that provider
        ↓
Cloud provider executes → Resources created
```

### Why Terraform over cloud-specific tools?

| Tool | Works On | Problem |
|------|----------|---------|
| AWS CloudFormation | AWS only | Useless on Azure/GCP |
| Azure ARM Templates | Azure only | Useless on AWS/GCP |
| **Terraform** | Any provider | ✅ One tool for all |

### Advantages of Terraform:
- **Track infrastructure** — state file shows all created resources
- **Automate changes** — no manual console logins
- **Collaborate** — store `.tf` files in Git for peer reviews
- **Standardized config** — same HCL syntax for any cloud

---

## 2. 🔁 Terraform Lifecycle

```
Write .tf files  →  terraform init  →  terraform plan  →  terraform apply  →  terraform destroy
```

| Command | Purpose |
|---------|---------|
| `terraform init` | Initialize provider plugins (reads `required_providers` block) |
| `terraform plan` | Dry run — shows what WILL happen without executing |
| `terraform apply` | Execute changes on the cloud provider |
| `terraform destroy` | Delete all resources defined in config |

> **Re-run `terraform init`** whenever you add a new provider or change backend config.

---

## 3. 📦 Installation

### Ubuntu/Linux:
```bash
sudo apt update && sudo apt install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install terraform
```

### Mac:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Already installed but old version? Upgrade:
brew upgrade hashicorp/tap/terraform
```

### Verify:
```bash
terraform --version     # Check version (should be >= 1.2.0)
terraform help          # List all commands
```

> ✅ Always use your native package manager (apt/brew/yum). Avoid manual `curl` installs — path issues are common.

---

## 4. 🔐 Authenticating Terraform with AWS

Terraform uses your already-configured AWS CLI credentials. No need to put keys in `.tf` files.

```bash
# Configure AWS CLI first:
aws configure
# AWS Access Key ID: <your-key>
# AWS Secret Access Key: <your-secret>
# Default region: us-west-2
# Default output format: json

# Test if auth works:
aws s3 ls
# If this works → terraform will work too ✅
```

> For **Azure**: configure Azure Service Principal first.
> For **GCP**: configure `gcloud auth application-default login`.

---

## 5. 📝 Writing Your First Terraform File

### File: `main.tf`

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

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformDemo"
  }
}
```

### File Structure Explained:

```
terraform {}          ← Terraform block — provider + version requirements
provider "aws" {}     ← Which cloud and region to use
resource "..." {}     ← What to create (differs per project)
```

> The `terraform {}` and `provider {}` blocks are almost always the same.
> Only the `resource {}` blocks change per project.
> **Always refer to Terraform docs** for resource syntax — nobody memorizes 200+ AWS service configs.

---

## 6. ▶️ Running Your First Project

```bash
# Step 1 — Initialize
terraform init
# Downloads AWS provider plugin

# Step 2 — Dry run
terraform plan
# Shows: what will be created, changed, or destroyed

# Step 3 — Apply
terraform apply
# Type 'yes' to confirm → EC2 instance is created on AWS

# Step 4 — Destroy (cleanup)
terraform destroy
# Type 'yes' → all resources deleted
```

### Sample `terraform plan` output:
```
Plan: 1 to add, 0 to change, 0 to destroy.

+ resource "aws_instance" "app_server" {
    ami           = "ami-830c94e3"
    instance_type = "t2.micro"
    tags = { Name = "TerraformDemo" }
  }
```

---

## 7. 📄 variables.tf and outputs.tf

### Why use them?

> Never hardcode values like AMI IDs, instance types, CIDR blocks directly in `main.tf`.
> Put them in `variables.tf` so changes are made in one place, not scattered across resource files.

### `variables.tf` (Input Variables):
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI image ID"
  default     = "ami-830c94e3"
}
```

### Referencing in `main.tf`:
```hcl
resource "aws_instance" "app_server" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

### `outputs.tf` (Output Values):
```hcl
output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.app_server.public_ip
}

output "instance_id" {
  description = "Instance ID"
  value       = aws_instance.app_server.id
}
```

> Outputs are shown in the terminal after `terraform apply`.
> Critical for users who don't have AWS console access and are running Terraform through Jenkins pipelines.

---

## 8. 🗂️ Terraform State File

### What is it?
After `terraform apply`, Terraform auto-creates `terraform.tfstate` — a JSON file that tracks **everything** it created.

```json
{
  "resources": [
    {
      "type": "aws_instance",
      "name": "app_server",
      "instances": [
        {
          "attributes": {
            "id": "i-0abc123def456789",
            "instance_type": "t2.micro",
            "availability_zone": "us-west-2a",
            ...
          }
        }
      ]
    }
  ]
}
```

### Why is it critical?
- Terraform uses state to know **what already exists**
- Without it, Terraform cannot update or destroy resources
- It tracks ALL sensitive info: VPC IDs, KMS keys, IP addresses, credentials

### ❌ What NOT to do with State File:

| Anti-Pattern | Why It's Wrong |
|-------------|----------------|
| Store in Git | Exposes secrets; can't merge multiple copies |
| Store on local machine | Not shared; each engineer gets a different state |
| Manually edit the state | Corrupts Terraform's understanding of infrastructure |
| Give write permissions to everyone | Only Terraform should write to it |

### ✅ State File Best Practices:

1. **Always use remote backend** (S3 / Azure Blob) — never local
2. **Never commit to Git** — add `*.tfstate` to `.gitignore`
3. **Never manually edit** — use `terraform state` commands if needed
4. **Isolate per environment** — separate state for dev, staging, prod
5. **Enable S3 versioning** — allows rollback if state is corrupted
6. **Use DynamoDB locking** — prevents parallel execution conflicts

---

## 9. ☁️ Remote Backend — S3 + DynamoDB Setup

### Why Remote Backend?

```
Without remote backend:
  Engineer A runs apply → state saved on A's laptop
  Engineer B runs apply → state saved on B's laptop
  Both states are out of sync → disaster 💥

With S3 remote backend:
  Both A and B read/write to same S3 state file
  DynamoDB locks state during apply → no conflicts ✅
```

### Step 1 — Create S3 Bucket + DynamoDB Table:
```hcl
# In a separate main.tf (run this first):

resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-bucket"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "state_versioning" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Step 2 — Add Backend Config to `main.tf`:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-lock"
    encrypt        = true
  }
}
```

### Re-initialize after adding backend:
```bash
terraform init
# Terraform migrates local state to S3 automatically
```

---

## 10. 🏗️ Ideal Terraform Setup (Production)

```
┌─────────────────────────────────────────────────────┐
│  Git Repository                                      │
│  (main.tf, variables.tf, outputs.tf — version ctrl) │
└──────────────────────┬──────────────────────────────┘
                       │ PR trigger / merge
                       ▼
┌─────────────────────────────────────────────────────┐
│  CI/CD Pipeline (Jenkins / GitHub Actions)           │
│  - terraform plan on PR                              │
│  - terraform apply on merge to main                  │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌──────────────┐    ┌──────────────────────────────────┐
│  S3 Bucket   │◄───│  Terraform Execution              │
│  (State file)│    └──────────────────┬───────────────┘
└──────────────┘                       │ API calls
│  DynamoDB    │                       ▼
│  (Locking)   │    ┌──────────────────────────────────┐
└──────────────┘    │  AWS / Azure / GCP               │
                    │  (Resources created)              │
                    └──────────────────────────────────┘
                                       │ outputs
                                       ▼
                    ┌──────────────────────────────────┐
                    │  outputs.tf                       │
                    │  (IPs, IDs, endpoints printed)    │
                    └──────────────────────────────────┘
```

> **Rule:** Never run `terraform apply` manually from a local machine in production. Always go through CI/CD.

---

## 11. 🧩 Terraform Modules

### What are modules?
Reusable Terraform configurations — write once, use across multiple teams or environments.

### When to use?
- Same infrastructure pattern needed in dev/staging/prod
- Multiple teams need the same base setup
- You want to enforce org-wide standards

### Example — Using a module:
```hcl
module "backend_setup" {
  source = "github.com/your-org/terraform-backend-module"

  bucket_name    = "team-a-state"
  dynamodb_table = "team-a-lock"
  region         = "us-east-1"
}
```

> Instead of every team writing their own S3 + DynamoDB setup, they reference the shared module. One fix = fixed for everyone.

---

## 12. ⚠️ Problems / Disadvantages of Terraform

| Problem | Details |
|---------|---------|
| **State file = single point of failure** | If corrupted/lost → Terraform loses track of all infrastructure |
| **Manual cloud changes break Terraform** | Someone changes EC2 in AWS console → state is out of sync. Fix: `terraform refresh` |
| **Not GitOps-friendly** | Terraform doesn't auto-detect or auto-correct infrastructure drift |
| **Complex at scale** | Managing 100s of accounts/environments requires extra tooling (Terragrunt, Atlantis) |
| **Not a config management tool** | Using Terraform to install software on servers is wrong — that's Ansible's job |

> These are **gold** in interviews. Interviewers are far more impressed when you can discuss real limitations than just list features.

---

## 13. 🎯 Interview Questions

> *From a 10-year hiring manager's perspective — these are the exact questions asked at mid-to-senior DevOps roles.*

---

### 🟢 Beginner

**Q1. What are the four Terraform commands and what does each do?**

> `init` — downloads provider plugins and initializes backend.
> `plan` — dry run showing what will be created/changed/destroyed.
> `apply` — executes the plan and creates real resources.
> `destroy` — deletes all resources defined in config.

---

**Q2. What is a Terraform provider? Give examples.**

> A provider is a plugin that lets Terraform talk to a specific platform's API. Examples: `hashicorp/aws`, `hashicorp/azurerm`, `hashicorp/google`, `hashicorp/kubernetes`. Declared in `required_providers` block; downloaded during `terraform init`.

---

**Q3. What is the Terraform state file and why is it important?**

> `terraform.tfstate` is Terraform's memory — it tracks every resource Terraform has created. It stores resource IDs, configurations, and metadata. Without it, Terraform cannot know what already exists and cannot update or destroy resources correctly.

---

**Q4. What is `terraform plan` and why should you always run it before `apply`?**

> `terraform plan` is a dry run — it shows exactly what will change without making any changes. Always run it to catch mistakes, review unintended changes, get team approval, and understand cost implications before touching real infrastructure.

---

### 🟡 Intermediate

**Q5. Describe your Terraform setup in your organization.**

> Strong answer: "We store all `.tf` files in a Git repository with branch protection. No one runs `terraform apply` manually — it runs through a Jenkins pipeline (plan on PR, apply on merge). The state file is stored in an S3 bucket with versioning enabled, and we use a DynamoDB table for state locking to prevent parallel execution conflicts. State files are isolated per environment — dev, staging, and prod each have their own state."

---

**Q6. Why should you never store the Terraform state file in Git?**

> Two reasons:
> 1. **Security** — state files contain sensitive data: KMS keys, VPC IDs, passwords, IP addresses. Anyone with repo access could read it.
> 2. **Team collaboration** — if two engineers each clone the repo and run apply, they get separate local state files that can never be reliably merged. Infrastructure tracking breaks completely.

---

**Q7. What is the purpose of DynamoDB when used with S3 for Terraform state?**

> DynamoDB provides **state locking**. When one `terraform apply` starts, it locks the state in DynamoDB. If a second person tries to apply simultaneously, they get a lock error and must wait. Without this, parallel execution causes conflicting changes and potential state file corruption.

---

**Q8. What are Terraform modules and when would you use them?**

> Modules are reusable, self-contained Terraform configurations stored separately and referenced by other configs. Use them when the same infrastructure pattern is needed across multiple teams or environments. Instead of copy-pasting 50 lines in 10 places, write a module once — one fix propagates everywhere.

---

**Q9. What is the difference between `variables.tf` and `outputs.tf`?**

> `variables.tf` declares input parameters your config accepts — things like instance type, region, AMI ID. It keeps hardcoded values out of resource files.
> `outputs.tf` defines what information Terraform should print after resources are created — like EC2 public IP or RDS endpoint. Crucial for users running Terraform through pipelines without AWS console access.

---

**Q10. When do you need to re-run `terraform init`?**

> You must re-run `terraform init` when:
> - You add a new provider to `required_providers`
> - You change the backend configuration (e.g., switch from local to S3)
> - You upgrade provider versions
> - You add or update modules

---

### 🔴 Advanced / Scenario-Based

**Q11. What are the biggest problems with Terraform at scale?**

> 1. State file is a single point of failure — corruption = infrastructure blindness
> 2. Manual console changes cause state drift — Terraform doesn't auto-detect or correct
> 3. Not GitOps-friendly — no native bi-directional sync between Git and cloud reality
> 4. Becomes complex with many environments — requires tools like Terragrunt to manage DRY configs
> 5. Misuse as a config management tool — using it to install software instead of Ansible leads to messy, hard-to-maintain code

---

**Q12. Someone on your team made a direct change in the AWS console that conflicts with your Terraform state. What do you do?**

> Run `terraform refresh` — this reads the actual state of all resources from AWS and updates the state file to match reality.
> Then run `terraform plan` to see the diff between your config and the refreshed state.
> Decide: either update your `.tf` files to match the manual change (and import it), or apply Terraform to revert the manual change.
> Long-term fix: use AWS Config rules or SCPs to prevent manual console changes on Terraform-managed resources.

---

**Q13. How do you isolate Terraform state per environment and why?**

> Create separate state files for dev, staging, and prod by using different S3 key paths:
> ```hcl
> # dev backend
> key = "dev/terraform.tfstate"
>
> # prod backend
> key = "prod/terraform.tfstate"
> ```
> Why: reduces blast radius. A corrupted or accidentally destroyed prod state doesn't affect dev. Also allows different teams to manage different environments independently.

---

**Q14. How does Terraform authenticate to AWS without credentials in the `.tf` files?**

> Terraform uses the same credential chain as the AWS CLI:
> 1. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
> 2. AWS CLI config (`~/.aws/credentials`) set via `aws configure`
> 3. IAM role attached to the EC2 instance / container running Terraform
>
> In CI/CD pipelines, best practice is to use IAM roles (not hardcoded keys) so no secrets are stored anywhere.

---

**Q15. Your team wants to add an Application Load Balancer to existing Terraform infrastructure. Walk through your approach.**

> 1. Go to [Terraform AWS provider docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb) and find `aws_lb` resource syntax
> 2. Add the resource block to your existing `.tf` files (or a new `alb.tf` file)
> 3. Move any hardcoded values (security group IDs, subnet IDs) to `variables.tf`
> 4. Add ALB DNS name to `outputs.tf` so the team gets the endpoint
> 5. Run `terraform plan` — review changes (new resources shown in green)
> 6. Create a PR, get peer review of the plan output
> 7. Merge → CI/CD runs `terraform apply`

---

## 14. 📁 Recommended File Structure

```
your-terraform-project/
├── main.tf           # Provider + resource definitions
├── variables.tf      # Input variable declarations
├── outputs.tf        # Output value declarations
├── terraform.tfvars  # Actual variable values (don't commit secrets)
├── backend.tf        # Remote backend config (S3 + DynamoDB)
└── README.md         # What this project creates
```

### For multi-environment orgs:
```
terraform/
├── modules/
│   ├── ec2/
│   ├── rds/
│   └── vpc/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
```

---

## 15. 📚 Resources

- 🔗 [Terraform Official Docs](https://developer.hashicorp.com/terraform/docs)
- 🔗 [Terraform AWS Provider Registry](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- 🔗 [Abhishek's Terraform Zero to Hero (GitHub)](https://github.com/abhishekf5/terraform-zero-to-hero)
- 🔗 [HashiCorp Learn — Terraform Tutorials](https://developer.hashicorp.com/terraform/tutorials)

---

## ✅ Practice Checklist

- [ ] Install Terraform on your machine (verify with `terraform --version`)
- [ ] Configure AWS CLI (`aws configure`)
- [ ] Clone the GitHub repo and navigate to `AWS/local-state`
- [ ] Write `main.tf` with provider block + EC2 resource
- [ ] Run `terraform init` → `terraform plan` → `terraform apply`
- [ ] Verify EC2 instance created in AWS console
- [ ] Add `outputs.tf` to print public IP after apply
- [ ] Add `variables.tf` and replace hardcoded values
- [ ] Run `terraform destroy` to clean up
- [ ] Create S3 bucket + DynamoDB table for remote backend
- [ ] Update `main.tf` with backend config and re-run `terraform init`
- [ ] Can answer all 15 interview questions confidently
