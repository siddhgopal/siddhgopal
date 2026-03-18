# 🤖 Common Errors Faced by DevOps Engineers in Ansible

> Ansible is a popular automation tool, but like any tool, it comes with its own set of challenges. Below is a list of common errors DevOps engineers encounter while using Ansible, along with potential causes and solutions.

---

## 📋 Table of Contents

| # | Error | Quick Fix |
|---|-------|-----------|
| 1 | [Permission Denied (SSH)](#1-permission-denied-error-ssh-issues) | Check SSH key + permissions |
| 2 | [Failed to Find Handler](#2-failed-to-find-handler-error) | Check handler definition |
| 3 | [No Host Matching](#3-no-host-matching-error) | Verify inventory file |
| 4 | [Failed to Load (Module Not Found)](#4-failed-to-load-error-module-not-found) | Install missing module |
| 5 | [Unreachable Hosts](#5-unreachable-hosts-error) | Check network + firewall |
| 6 | [Module Execution Failed](#6-module-execution-failed-error) | Check module arguments |
| 7 | [Incomplete List of Hosts](#7-incomplete-list-of-hosts-error) | Fix inventory config |
| 8 | [Unexpected TTY](#8-unexpected-tty-error) | Add TTY allocation |
| 9 | [Jinja2 Template Syntax Error](#9-jinja2-template-syntax-error) | Fix template syntax |
| 10 | [Tasks Failed with Changed = False](#10-tasks-failed-with-changed--false) | Check task conditions |

---

## 1. `Permission Denied` Error (SSH Issues)

### 🔴 What it is
Ansible cannot SSH into the target machine, and you get a `Permission Denied` error.

### ⚠️ Possible Causes
- Incorrect SSH credentials (username, password, or key)
- SSH key not added to the remote server or incorrect permissions
- Misconfigured SSH configuration or firewall blocking connections

### ✅ How to Fix

```bash
# Fix SSH key permissions
chmod 600 ~/.ssh/id_rsa

# Test SSH connection manually
ssh -i ~/.ssh/id_rsa user@target-host

# Run Ansible with verbose mode to debug
ansible all -m ping -vvv

# Check SSH key in inventory
ansible all -m ping --private-key=~/.ssh/id_rsa -u ubuntu
```

```ini
# inventory.ini — correct SSH config
[webservers]
192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

> 💡 **Tip:** Always ensure SSH key has `chmod 600` permissions. Verify the SSH configuration on the remote machine.

---

## 2. `Failed to Find Handler` Error

### 🔴 What it is
Ansible can't find the specified handler when trying to notify it.

### ⚠️ Possible Causes
- The handler doesn't exist in the playbook or role
- The task is notified, but the handler isn't defined
- Spelling or indentation mismatch

### ✅ How to Fix

```yaml
# Wrong - handler name mismatch
tasks:
  - name: Install nginx
    apt:
      name: nginx
    notify: Restart Nginx   # Capital N

handlers:
  - name: restart nginx     # Lowercase n — MISMATCH!
    service:
      name: nginx
      state: restarted

# Correct - names must match exactly
tasks:
  - name: Install nginx
    apt:
      name: nginx
    notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

> 💡 **Tip:** Handler names are case-sensitive. Double-check spelling and indentation.

---

## 3. `No Host Matching` Error

### 🔴 What it is
Ansible cannot find the target host specified in your inventory.

### ⚠️ Possible Causes
- Incorrect host group or host name in your inventory file
- Typo in the playbook or inventory file
- Missing host in the inventory file

### ✅ How to Fix

```bash
# Verify inventory hosts
ansible-inventory --list

# Test connectivity to all hosts
ansible all -m ping

# Check specific group
ansible webservers -m ping

# Run with custom inventory file
ansible all -i inventory.ini -m ping
```

```ini
# inventory.ini — correct format
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20
```

> 💡 **Tip:** Use `ansible-inventory --list` to verify your inventory is being parsed correctly.

---

## 4. `Failed to Load` Error (Module Not Found)

### 🔴 What it is
Ansible can't find the specified module.

### ⚠️ Possible Causes
- The module isn't installed or available in the Ansible environment
- Incorrect module name or version

### ✅ How to Fix

```bash
# Install missing collection/module
ansible-galaxy collection install community.general

# Install specific module
ansible-galaxy collection install amazon.aws

# List installed collections
ansible-galaxy collection list

# Check Ansible version
ansible --version
```

```yaml
# Use fully qualified collection name (FQCN)
- name: Create S3 bucket
  amazon.aws.s3_bucket:
    name: my-bucket
    state: present
```

> 💡 **Tip:** Always use the fully qualified collection name (FQCN) to avoid module conflicts.

---

## 5. `Unreachable Hosts` Error

### 🔴 What it is
Ansible reports some hosts as unreachable.

### ⚠️ Possible Causes
- The target host is down or unreachable due to network issues
- Incorrect inventory configuration
- Firewall or security groups blocking Ansible's connection

### ✅ How to Fix

```bash
# Test basic connectivity
ping target-host

# Test SSH port is open
nc -zv target-host 22

# Run Ansible with verbose for details
ansible all -m ping -vvv

# Check with specific user
ansible all -m ping -u ubuntu
```

```ini
# inventory.ini — add connection settings
[webservers]
192.168.1.10 ansible_user=ubuntu ansible_port=22 ansible_connection=ssh
```

> 💡 **Tip:** Ensure firewall/security groups allow port 22 from your Ansible control node.

---

## 6. `Module Execution Failed` Error

### 🔴 What it is
A specific module fails to execute on the target machine.

### ⚠️ Possible Causes
- Incorrect module arguments or syntax
- Missing dependencies on the target machine
- Permissions issues on the target system

### ✅ How to Fix

```bash
# Run with verbose to see full error
ansible-playbook playbook.yml -vvv

# Check module documentation
ansible-doc apt
ansible-doc service

# Test module directly
ansible webservers -m apt -a "name=nginx state=present" --become
```

```yaml
# Correct module usage example
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes
  become: yes          # Run as sudo
  become_user: root
```

> 💡 **Tip:** Use `ansible-doc <module-name>` to check correct module syntax and required arguments.

---

## 7. `Incomplete List of Hosts` Error

### 🔴 What it is
Ansible cannot execute tasks because it can't find all the hosts listed in the inventory.

### ⚠️ Possible Causes
- Dynamic inventories that are incorrectly configured
- Inventory file contains unreachable or missing hosts

### ✅ How to Fix

```bash
# Test all hosts are reachable
ansible all -m ping

# Check dynamic inventory script
ansible-inventory -i dynamic_inventory.py --list

# Run with specific subset of hosts
ansible-playbook playbook.yml --limit webservers

# Skip unreachable hosts
ansible-playbook playbook.yml --ignore-unreachable
```

> 💡 **Tip:** Use `--limit` flag to run playbooks on a subset of hosts during troubleshooting.

---

## 8. `Unexpected TTY` Error

### 🔴 What it is
Ansible runs into an issue when executing commands with elevated privileges (sudo) and can't allocate a pseudo-terminal (TTY).

### ⚠️ Possible Causes
- The user doesn't have permission to run commands with sudo
- The system requires a TTY for sudo but Ansible isn't set up to allocate one

### ✅ How to Fix

```bash
# Force TTY allocation
ansible all -m ping --ssh-extra-args='-t'
```

```ini
# inventory.ini — add TTY flag
[webservers]
192.168.1.10 ansible_ssh_extra_args='-t'
```

```ini
# /etc/sudoers — disable requiretty (on target machine)
# Comment out this line:
# Defaults requiretty

# Or add exception for ansible user:
Defaults:ansible !requiretty
```

```yaml
# ansible.cfg
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -t
```

> 💡 **Tip:** The cleanest fix is to disable `requiretty` in `/etc/sudoers` on target machines.

---

## 9. `Jinja2 Template Syntax Error`

### 🔴 What it is
Ansible throws an error when there's an issue with Jinja2 templating in your playbook or role.

### ⚠️ Possible Causes
- Incorrect use of Jinja2 syntax (missing brackets or mismatched quotations)
- Undefined variables used in templates

### ✅ How to Fix

```yaml
# Wrong - missing quotes around variable
- name: Create directory
  file:
    path: /var/{{ app_name      # Missing closing braces!
    state: directory

# Correct - proper Jinja2 syntax
- name: Create directory
  file:
    path: "/var/{{ app_name }}"
    state: directory
```

```bash
# Check if variable is defined
ansible all -m debug -a "var=app_name"

# Run with verbose to see template errors
ansible-playbook playbook.yml -vvv
```

```yaml
# Always set default values for variables
vars:
  app_name: "{{ app_name | default('myapp') }}"
  app_port: "{{ app_port | default(8080) }}"
```

> 💡 **Tip:** Always use `| default('value')` filter to handle undefined variables gracefully.

---

## 10. `Tasks Failed` with `Changed = False`

### 🔴 What it is
Ansible shows that a task has failed, but the state is set to `Changed = False`.

### ⚠️ Possible Causes
- The task did not execute as expected due to conditions in the playbook
- The `when` condition set for the task is not being met

### ✅ How to Fix

```yaml
# Wrong - condition never met
- name: Restart service
  service:
    name: nginx
    state: restarted
  when: ansible_os_family == "Redhat"  # Typo! Should be "RedHat"

# Correct - check exact value
- name: Restart service
  service:
    name: nginx
    state: restarted
  when: ansible_os_family == "RedHat"
```

```bash
# Debug variable values
ansible all -m debug -a "var=ansible_os_family"

# Run with check mode to test conditions
ansible-playbook playbook.yml --check

# Run with verbose
ansible-playbook playbook.yml -vvv
```

```yaml
# Add debug task to troubleshoot conditions
- name: Debug condition
  debug:
    msg: "OS Family is {{ ansible_os_family }}"
```

> 💡 **Tip:** Use `ansible all -m debug -a "var=variable_name"` to check exact variable values before writing conditions.

---

## 🔧 Quick Reference Commands

```bash
# Most used Ansible commands
ansible all -m ping                          # Test connectivity
ansible all -m setup                         # Gather facts
ansible-playbook playbook.yml                # Run playbook
ansible-playbook playbook.yml --check        # Dry run
ansible-playbook playbook.yml -vvv           # Verbose mode
ansible-playbook playbook.yml --limit host1  # Run on specific host
ansible-playbook playbook.yml --tags deploy  # Run specific tags
ansible-playbook playbook.yml --skip-tags setup  # Skip tags
ansible-inventory --list                     # List all hosts
ansible-galaxy collection install <name>     # Install collection
ansible-doc <module>                         # Module documentation
ansible-vault encrypt secrets.yml            # Encrypt secrets file
ansible-vault decrypt secrets.yml            # Decrypt secrets file
```

---

## 👨‍💻 Author

**Siddhgopal Soni** — DevOps Engineer | AWS · Kubernetes · Terraform · CI/CD

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/siddhgopal-soni-010846b8)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)](https://github.com/siddhgopal)

---

⭐ *If this helped you, give this repo a star!*
