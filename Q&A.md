
# Ansible Interview Questions & Answers

A comprehensive list of frequently asked Ansible interview questions with answers and examples for beginners and experienced professionals.

## Table of Contents

1. [Basics](#basics)  
2. [Intermediate Concepts](#intermediate-concepts)  
3. [Advanced Concepts](#advanced-concepts)  
4. [Scenario-Based Questions](#scenario-based-questions)  
5. [Practical / Hands-On Questions](#practical-hands-on-questions)  

---

## Basics

**Q1: What is Ansible?**  
**A:** Ansible is an open-source IT automation tool used for configuration management, application deployment, task automation, and orchestration. It is agentless and uses YAML-based playbooks.

**Q2: What are the main benefits of using Ansible?**  
- Agentless architecture (uses SSH/WinRM)  
- Human-readable YAML syntax  
- Idempotent execution  
- Works across multiple platforms (Linux, Windows, cloud, networking)  
- Integrates with CI/CD pipelines  

**Q3: What is a control node and managed node?**  
- **Control Node:** Machine where Ansible is installed and playbooks are executed.  
- **Managed Node:** Target machines that Ansible manages over SSH/WinRM.  

**Q4: What is an inventory in Ansible?**  
Inventory is a list of managed nodes grouped logically. Example:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
````

**Q5: What is idempotency in Ansible?**
Idempotency means running a playbook multiple times results in the same state without unnecessary changes.

**Q6: How does Ansible communicate with nodes?**
Uses SSH for Linux/Unix nodes and WinRM for Windows nodes.

**Q7: What is a playbook in Ansible?**
Playbooks are YAML files that define tasks to automate on managed nodes. They can include tasks, handlers, roles, and variables.

**Q8: Mention some common use cases of Ansible.**

* Configuration management
* Application deployment
* Orchestration
* Provisioning cloud infrastructure
* Security automation

---

## Intermediate Concepts

**Q9: What is a module in Ansible?**
Modules are reusable scripts executed on managed nodes to perform specific tasks (e.g., `yum`, `apt`, `service`, `copy`).

**Q10: What are facts in Ansible?**
Facts are variables automatically gathered about managed nodes (IP, OS, memory) using `gather_facts: yes`.

**Q11: How do you define variables in Ansible?**
Variables can be defined in playbooks (`vars`), external files (`vars_files`), or inventory (`host_vars` / `group_vars`).

**Q12: What are handlers in Ansible?**
Handlers are tasks triggered by `notify` statements, usually used for restarting services after configuration changes.

**Q13: Explain conditionals in Ansible.**

```yaml
- name: Install nginx only on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"
```

**Q14: How do you run a subset of tasks using tags?**
Use `tags` in tasks and run with `--tags`:

```bash
ansible-playbook site.yml --tags "install"
```

**Q15: Explain loops in Ansible.**

```yaml
- name: Install multiple packages
  package:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - vim
```

**Q16: Difference between `changed_when` and `failed_when`?**

* `changed_when`: Force a task to report as changed.
* `failed_when`: Define custom conditions to mark a task as failed.

**Q17: How do you debug variables in Ansible?**

```yaml
- debug:
    var: my_variable
```

**Q18: How do you skip tasks?**
Use `when: false` or `--skip-tags`. Example:

```bash
ansible-playbook site.yml --skip-tags "restart"
```

---

## Advanced Concepts

**Q19: What is Ansible Vault?**
A tool to encrypt sensitive data like passwords or keys. Commands: `ansible-vault create`, `encrypt`, `decrypt`.

**Q20: What are roles in Ansible?**
Roles are structured ways to organize tasks, handlers, variables, templates, and files for reusability.

**Q21: What is a block in Ansible?**

```yaml
- block:
    - command: /bin/false
  rescue:
    - debug: msg="Command failed"
  always:
    - debug: msg="Task completed"
```

**Q22: How do you use templates in Ansible?**

```yaml
- template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

**Q23: What are lookup plugins?**
Plugins retrieve external data like files, environment variables, or passwords. Example: `lookup('file', 'myfile.txt')`.

**Q24: Explain dynamic inventory.**
Dynamic inventory uses scripts or cloud plugins to fetch nodes automatically (AWS EC2, Azure, GCP).

**Q25: What are collections in Ansible?**
Collections are packaged sets of roles, modules, and plugins. Example: `community.aws.ec2`.

**Q26: Example of creating an EC2 instance using a collection:**

```yaml
- name: Launch EC2 instance
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Launch instance
      community.aws.ec2_instance:
        name: "my-test-instance"
        key_name: my-key
        instance_type: t2.micro
        image_id: ami-0abcd1234
        wait: yes
        region: us-east-1
```

**Q27: What is Ansible Galaxy?**
Galaxy is a repository for sharing roles and collections. Install with `ansible-galaxy install <role>`.

**Q28: How do you manage secrets in Ansible Vault?**
Encrypt files: `ansible-vault encrypt secrets.yml`
Decrypt: `ansible-vault decrypt secrets.yml`

**Q29: Explain fact caching.**
Stores gathered facts to avoid repeated collection, improving performance.

**Q30: Difference between static and dynamic inventory?**

* Static: Manually defined hosts in a file.
* Dynamic: Automatically fetched from scripts or cloud APIs.

---

## Scenario-Based Questions

**Q31: Deploy an application to multiple servers?**
Use a playbook targeting `[appservers]` group, include tasks for installing dependencies, copying code, and restarting services.

**Q32: Ensure idempotency in scripts?**
Use modules instead of raw shell commands; modules check state before making changes.

**Q33: Handle failures during deployment?**
Use `block`, `rescue`, and `always` for error handling.

**Q34: Perform rolling updates?**
Use `serial` keyword to update hosts in batches:

```yaml
- hosts: webservers
  serial: 2
```

**Q35: Run only specific tasks?**
Use tags: `--tags` or `--skip-tags`.

**Q36: Provision cloud instances dynamically?**
Use dynamic inventory or cloud collections in playbooks.

**Q37: Manage configuration drift?**
Schedule playbooks or use Ansible Tower/AWX to ensure nodes remain in desired state.

---

## Practical / Hands-On Questions

**Q38: Check host availability:**

```yaml
- ping:
```

**Q39: Install multiple packages in one task:**

```yaml
- package:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
```

**Q40: Copy a file to remote servers:**

```yaml
- copy:
    src: /tmp/config.txt
    dest: /etc/config.txt
```

**Q41: Restart a service only when config changes:**

```yaml
- copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- service:
    name: nginx
    state: restarted
```

**Q42: Use `register` and `debug` to inspect output:**

```yaml
- command: ls /tmp
  register: output
- debug:
    var: output.stdout_lines
```

**Q43: Include external roles in playbooks:**

```yaml
- hosts: all
  roles:
    - geerlingguy.nginx
```

**Q44: Encrypt passwords with Vault:**

```bash
ansible-vault encrypt_string 'mypassword' --name 'db_password'
```

**Q45: Create a directory on managed nodes:**

```yaml
- file:
    path: /opt/mydir
    state: directory
```

**Q46: Run playbooks in check mode:**

```bash
ansible-playbook site.yml --check
```

**Q47: Increase verbosity for debugging:**

```bash
ansible-playbook site.yml -vvv
```

**Q48: Remove a package:**

```yaml
- package:
    name: apache2
    state: absent
```

**Q49: Gather only specific facts:**

```yaml
gather_facts: yes
gather_subset:
  - network
  - hardware
```

**Q50: Loop over a list of users to create accounts:**

```yaml
- user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie
```

