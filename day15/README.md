# Ansible - Day 15 | Complete DevOps Course
> 📺 [Watch the Video](https://www.youtube.com/watch?v=Z6T2r3Xhk5k&list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&index=18) | By Abhishek

---

## 📌 What This Video Covers

- Installing Ansible
- Setting up Passwordless Authentication
- Ansible Ad-Hoc Commands
- Writing Your First Ansible Playbook
- Ansible Roles & Folder Structure

---

## 1. 🔧 Installation

Use your OS package manager (recommended over pip):

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# Mac
brew install ansible

# Windows
choco install ansible

# Verify
ansible --version
```

> **Tip:** Using pip sometimes requires manually adding Ansible to PATH. Package manager handles this automatically.

---

## 2. 🔑 Passwordless Authentication (SSH Key Setup)

Ansible requires passwordless SSH authentication to target servers.

### On Ansible (Control) Server:
```bash
ssh-keygen
# Keys stored at: ~/.ssh/id_rsa (private) and ~/.ssh/id_rsa.pub (public)

cat ~/.ssh/id_rsa.pub
# Copy this output
```

### On Target Server:
```bash
ssh-keygen
# Open authorized_keys and paste the Ansible server's public key
nano ~/.ssh/authorized_keys
# Paste the public key → Save
```

### Test:
```bash
ssh <target-server-private-ip>
# Should connect WITHOUT asking for password ✅
```

> **Rule:** Always use **private IP** when both servers are on the same VPC (AWS).
> 
> **To add more servers:** Just paste the same public key into their `~/.ssh/authorized_keys`.

---

## 3. 📋 Inventory File

The inventory file stores IP addresses of all target servers.

```bash
# Default location (can also be local)
/etc/ansible/hosts

# Example inventory file content:
172.31.62.28

# With grouping:
[webservers]
172.31.62.100

[dbservers]
172.31.62.200
```

> **Interview Q:** How do you run a playbook only on certain servers?
> **Answer:** Use groups in the inventory file and pass the group name instead of `all`.

---

## 4. ⚡ Ansible Ad-Hoc Commands

Use for **one or two quick tasks** — no need to write a playbook.

```bash
ansible -i inventory all -m shell -a "touch /tmp/devops_class"
#        ^inventory  ^hosts ^module ^arguments
```

### Common Examples:
```bash
# Create a file on target
ansible -i inventory all -m shell -a "touch /tmp/testfile"

# Check CPU count
ansible -i inventory all -m shell -a "nproc"

# Check disk usage
ansible -i inventory all -m shell -a "df -h"

# Copy a file
ansible -i inventory all -m copy -a "src=/local/file dest=/remote/path"

# Run only on a specific group
ansible -i inventory webservers -m shell -a "uptime"
```

> **Interview Q:** Difference between Ad-Hoc commands and Playbooks?
> **Answer:** Ad-hoc = one or two quick tasks from CLI. Playbook = multiple tasks in a structured YAML file.

---

## 5. 📝 Writing Your First Ansible Playbook

Playbooks are written in **YAML format**.

```yaml
---
- name: Install and Start Nginx
  hosts: all
  become: true   # Run as root (sudo)

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present   # 'present' = install, 'absent' = remove

    - name: Start Nginx
      service:
        name: nginx
        state: started
```

### Run the Playbook:
```bash
ansible-playbook -i inventory first-playbook.yaml

# With verbose/debug output:
ansible-playbook -i inventory first-playbook.yaml -vvv
```

### Playbook Execution Flow:
1. **Gathering Facts** — Ansible collects info about target servers
2. **Task 1** — Install Nginx
3. **Task 2** — Start Nginx

> **Tip:** Use `-vvv` (verbosity) to debug and understand what Ansible is doing internally.

---

## 6. 🗂️ Ansible Roles

Used when you have **complex playbooks with 50+ tasks** (e.g., configuring Kubernetes).

### Create a Role:
```bash
ansible-galaxy role init kubernetes
```

This creates the following folder structure:

```
kubernetes/
├── defaults/       # Default variable values
├── files/          # Static files to copy to target (e.g., index.html, certs)
├── handlers/       # Exception/event handling (e.g., restart service on failure)
├── meta/           # Metadata, licensing, author info
├── tasks/          # All tasks (main logic goes here → main.yml)
├── templates/      # Jinja2 templates for dynamic file generation
├── tests/          # Unit tests for the role
└── vars/           # Variable overrides (higher priority than defaults)
```

### Folder Explanations:

| Folder | Purpose |
|--------|---------|
| `tasks/` | Main task list (like what you'd write directly in a playbook) |
| `handlers/` | Triggered by events — e.g., restart nginx when config changes |
| `vars/` | Variables specific to the role |
| `defaults/` | Default values (lowest priority, easily overridden) |
| `files/` | Static files to be copied as-is to target servers |
| `templates/` | Dynamic files using Jinja2 templating (`{{ variable }}`) |
| `meta/` | Role metadata — author, license, dependencies |
| `tests/` | Test playbooks for the role |

### Parent Playbook Using a Role:
```yaml
---
- name: Configure Kubernetes
  hosts: all
  become: true
  roles:
    - kubernetes
```

> All task details go inside `kubernetes/tasks/main.yml` instead of the parent file.

---

## 7. 🎯 Key Interview Questions

| Question | Answer |
|----------|--------|
| What is Ansible? | Open-source configuration management tool that uses agentless architecture and SSH |
| What is an Inventory file? | File containing IP addresses/hostnames of target servers; supports grouping |
| Ad-Hoc vs Playbook? | Ad-hoc = quick CLI commands for 1-2 tasks; Playbook = YAML file for multiple tasks |
| What is `become: true`? | Escalates privilege to run as root (equivalent to `sudo`) |
| What is Gather Facts? | First automatic step — Ansible collects target server details before running tasks |
| How to group servers? | Use `[groupname]` syntax in inventory file |
| Why use Roles? | To organize complex playbooks into reusable, structured folders |
| Difference: `vars` vs `defaults`? | `vars` has higher priority; `defaults` can be easily overridden |
| What is Handlers? | Tasks triggered by events (e.g., restart nginx when config changes) |
| What is Ansible Galaxy? | Community hub for sharing roles; also used to init role structure |

---

## 8. 🏗️ Real-World Use Case (DevOps)

**Setting up Kubernetes on AWS:**

| Tool | Task |
|------|------|
| **Terraform** | Create 3 EC2 instances (infrastructure provisioning) |
| **Ansible** | Configure 1 as Master, 2 as Worker nodes |

> Rule of thumb: Use **Terraform** for infrastructure creation, **Ansible** for configuration management.

---

## 🚀 Practice Checklist

- [ ] Install Ansible on a Linux machine
- [ ] Launch 2 EC2 instances (or VMs)
- [ ] Set up passwordless SSH between them
- [ ] Create an inventory file
- [ ] Run an ad-hoc command to create a file on the target
- [ ] Write a playbook to install and start Nginx
- [ ] Verify Nginx is running on the target server
- [ ] Create a role using `ansible-galaxy role init`
- [ ] Explore the JBoss Standalone example from the GitHub repo
