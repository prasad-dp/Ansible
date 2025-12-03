

---

# **1. Basics and Architecture**

## **What is Ansible?**

Ansible is an open-source automation engine designed to simplify IT operations such as configuration management, application deployment, server provisioning, orchestration, and daily task automation. It enables administrators and DevOps engineers to manage a large number of servers efficiently and consistently from a central control node.

One of Ansible’s core strengths is its **agentless architecture**. This means you do not need to install any software or agent on the systems you manage. Instead, Ansible relies on widely used and secure communication channels such as **SSH** (for Linux/Unix systems) and **WinRM** (for Windows systems). This makes deployment simple and reduces overhead.

Ansible uses **YAML-based playbooks**, which are human-readable and easy to write, even for beginners. These playbooks define the desired state of your systems, and Ansible ensures those systems match that state reliably and repeatedly.

---

## **Use Cases of Ansible**

Ansible is a highly flexible automation tool and can be used across multiple areas of IT:

### **1. Configuration Management**

Ensure systems remain in a desired state:

* Installing packages
* Configuring services
* Managing files and users
* Setting up system parameters

### **2. Application Deployment**

Deploy applications across multiple servers consistently and rapidly.

### **3. Orchestration**

Coordinate tasks across multiple servers or services to perform complex workflows.

### **4. Provisioning**

Automate the creation and setup of:

* Virtual machines
* Cloud instances (AWS, Azure, GCP)
* Network devices

### **5. Continuous Delivery**

Automate integration and deployment pipelines as part of DevOps workflows.

### **6. Security Automation**

Automate critical tasks such as:

* Patching servers
* Managing users and permissions
* Enforcing security compliance
* Running vulnerability scans

---

## **Core Components of Ansible**

### **1. Control Node (Management Node)**

This is the machine where Ansible is installed.
You run commands such as:

```
ansible
ansible-playbook
```

The control node:

* Does not require powerful hardware
* Only needs Python installed
* Is the central point from where automation is executed

---

### **2. Managed Nodes (Targets)**

These are the servers or devices that Ansible manages.
Examples:

* Linux servers
* Windows servers
* Network devices (Cisco/Juniper)
* Cloud resources

Ansible connects to these systems using:

* **SSH** for Linux
* **WinRM** for Windows

Nothing needs to be installed on these systems — this is why Ansible is called **agentless**.

---

### **3. Inventory File**

The inventory is a file (INI or YAML format) that lists all the managed nodes.

Example **inventory.ini** file:

```ini
[webservers]
server1.example.com
server2.example.com

[databases]
db.example.com

[all:vars]
ansible_user=deployer
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Purpose of inventory:

* Group systems (webservers, databases, etc.)
* Assign variables to groups or hosts
* Target specific machines in playbooks

---

## **Installation Guide**

To begin using Ansible, you need to install it on your **control node**.

### **Prerequisites**

* Python 3
* pip (recommended)

---

### **Installation Methods**

#### **1. Using pip (Recommended)**

```bash
pip install ansible
```

---

#### **2. Debian/Ubuntu**

```bash
sudo apt update && sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

---

#### **3. Red Hat / CentOS / Fedora**

```bash
sudo dnf install epel-release -y  # Fedora, CentOS 8+
sudo yum install epel-release -y  # CentOS 7

sudo dnf install ansible -y       # Fedora, CentOS 8+
sudo yum install ansible -y       # CentOS 7
```

---

#### **4. macOS**

```bash
brew install ansible
```

---

### **Verify Installation**

```bash
ansible --version
```

This confirms that Ansible is installed successfully on your control node.

---

## **Generating an SSH Key Pair for Ansible**

To allow Ansible to connect securely and passwordlessly to your managed nodes, you need an SSH key pair.

### **Step 1: Generate Key Pair**

Run the following command:

```bash
ssh-keygen -t rsa -b 4096
```

* `-t rsa` → RSA key algorithm
* `-b 4096` → high-security 4096-bit key

Press **Enter** to accept the default file location:
`~/.ssh/id_rsa`

You may set a passphrase for extra security (recommended in production).

This generates two files:

* **id_rsa** → private key (must be kept secret)
* **id_rsa.pub** → public key (can be shared)

---

## **Copying Public Key to Managed Hosts**

This enables passwordless SSH login, required for Ansible.

---

### **Method 1: Using ssh-copy-id (Recommended)**

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip
```

You will be prompted for the remote user’s password once.

After this, you can log in:

```bash
ssh username@server_ip
```

---

### **Method 2: Manual Copy & Paste**

#### **Step 1: Display your public key**

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the output.

#### **Step 2: SSH into the managed host**

```bash
ssh username@server_ip
```

#### **Step 3: Create .ssh directory**

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

#### **Step 4: Add key to authorized_keys**

```bash
nano ~/.ssh/authorized_keys
```

Paste the key.

#### **Step 5: Set permissions**

```bash
chmod 600 ~/.ssh/authorized_keys
```

Now SSH login works without password.

---
Perfect — continuing with **Section 2: Core Building Blocks**, written in the **same detailed, beginner-friendly, full-explanation style**.

When we finish all 14 sections, I will assemble everything into **one single final README.md**.

---

# **2. Core Building Blocks**

Ansible is built on a set of fundamental components that work together to automate systems. Understanding these building blocks is essential before writing playbooks or managing servers. In this section, we explain inventories, playbooks, tasks, handlers, roles, and common modules — all in simple, clear language with examples.

---

# **2.1 Inventory Files (Static Inventory)**

The **inventory** is one of the most important parts of Ansible. It is a file that tells Ansible *which servers to manage* and *how to connect to them*.

You can write inventory files in:

* **INI format** (very common)
* **YAML format**

A basic inventory in INI format looks like this:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com

[all:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### **Why Inventories Are Useful**

* They group servers logically (web servers, DB servers, etc.)
* You can target specific groups in your playbooks (e.g., only database servers)
* You can assign variables at group or host level
* Makes automation scalable and organized

### **Host-Specific Variables Example**

```ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu
```

### **Grouping Servers Helps You Run Commands Like:**

```bash
ansible webservers -m ping
ansible databases -m shell -a "df -h"
```

---

# **2.2 Playbook Structure**

A **playbook** is the heart of Ansible automation.
It is a YAML file that describes:

* Which hosts to target
* Tasks to execute
* Variables to use
* Handlers to notify
* Roles to include

A simple playbook structure looks like this:

```yaml
---
- name: Install and start Apache web server
  hosts: webservers
  become: yes

  vars:
    package_name: httpd

  tasks:
    - name: Install Apache
      package:
        name: "{{ package_name }}"
        state: present

    - name: Start Apache
      service:
        name: "{{ package_name }}"
        state: started
        enabled: yes
```

---

## **Breakdown of Playbook Components**

### ✔ **Play**

A play connects a group of hosts to tasks.

### ✔ **hosts**

Defines which inventory group or host to run the tasks on.

### ✔ **tasks**

Each task calls a module that performs an action (install package, copy file, start service).

### ✔ **vars**

Variables used within the playbook.

### ✔ **handlers**

Special tasks triggered by `notify`.

### ✔ **roles**

Reusable components that organize tasks, handlers, vars, templates, etc.

---

# **2.3 Common Modules (With Simple Explanations and Examples)**

Modules are the “tools” Ansible uses to do work on managed nodes.

These modules cover:

* Package management
* File operations
* User management
* Services
* Debugging
* Commands

Below are the most common modules beginners use.

---

## **1. Package Management Modules**

### **package (generic)**

Works on all Linux distros.

```yaml
- name: Install curl
  package:
    name: curl
    state: present
```

### **apt (Debian/Ubuntu)**

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: latest
```

### **yum (CentOS/RHEL 7)**

```yaml
- name: Install Apache on CentOS
  yum:
    name: httpd
    state: present
```

### **dnf (CentOS 8+/Fedora)**

```yaml
- name: Remove MariaDB
  dnf:
    name: mariadb-server
    state: absent
```

---

## **2. Service Module**

Manages system services.

```yaml
- name: Start and enable Apache
  service:
    name: httpd
    state: started
    enabled: yes
```

---

## **3. File and Copy Modules**

### **copy module**

Copies files from control node → managed node.

```yaml
- name: Copy config file
  copy:
    src: files/config.txt
    dest: /etc/config.txt
    owner: root
    group: root
    mode: '0644'
```

### **file module**

Create, remove, or modify files/directories.

```yaml
- name: Create a directory
  file:
    path: /opt/myapp
    state: directory
    mode: '0755'
```

States include:

* `directory`
* `file`
* `absent`
* `touch`

---

## **4. user Module**

Creates or modifies users.

```yaml
- name: Create a user
  user:
    name: deployer
    state: present
    groups: sudo
```

---

## **5. ping Module**

Checks connectivity.

```yaml
- name: Test connectivity
  ping:
```

---

## **6. debug Module**

Prints messages or variable values.

```yaml
- name: Print message
  debug:
    msg: "Deployment complete!"
```

---

## **7. command and shell Modules**

### **command module**

Runs a command (does NOT use shell features).

```yaml
- name: Run uptime
  command: uptime
```

### **shell module**

Runs commands through the shell (supports pipes, redirects, etc.)

```yaml
- name: Run a shell pipeline
  shell: "cat /etc/passwd | grep root"
```
Perfect — continuing with **Section 3: Variables and Facts**, written in a detailed, beginner-friendly README-ready format.

---

# **3. Variables and Facts**

Variables in Ansible make your playbooks dynamic, reusable, and easier to manage. Rather than hardcoding values (like package names, file paths, usernames, ports, etc.), you can store them in variables and use them throughout your playbooks.

Facts, on the other hand, are pieces of information that Ansible automatically gathers about managed hosts (like OS type, IP address, memory, CPU count, distribution version, etc.). Facts help you write smarter automation that adapts to different environments.

This section explains both **variables** and **facts** with simple examples.

---

# **3.1 What Are Variables in Ansible?**

Variables are named values that you can reference anywhere in your playbook.
Examples include:

* Package names (`nginx`, `httpd`)
* Ports (`80`, `443`)
* File paths (`/var/www/html`)
* Usernames (`ansible`, `deployer`)
* Boolean values (`true`, `false`)

Using variables helps avoid repetition and makes your playbooks cleaner and more maintainable.

### **Basic Example of a Variable**

```yaml
vars:
  app_port: 8080
```

Use it like this:

```yaml
debug:
  msg: "Application will run on port {{ app_port }}"
```

---

# **3.2 Ways to Define Variables in Ansible**

Ansible offers multiple ways to define variables. The most common are:

### ✔ Playbook variables (`vars:` section)

### ✔ Vars files (`vars_files:`)

### ✔ Inventory variables

### ✔ Group and host-specific vars

### ✔ Extra vars (`--extra-vars` or `-e`)

We will explore each with examples.

---

# **3.2.1 Variables Inside a Playbook**

Define variables using the `vars:` keyword.

### **Example**

```yaml
---
- name: Setting basic variables
  hosts: all
  
  vars:
    web_package: nginx
    web_port: 80

  tasks:
    - name: Install web package
      package:
        name: "{{ web_package }}"
        state: present

    - name: Print the port
      debug:
        msg: "The web server listens on port {{ web_port }}"
```

---

# **3.2.2 External Variables Files (vars_files)**

Best for large or shared variables.

### **vars file: `web_vars.yml`**

```yaml
web_package: nginx
web_root: /var/www/html
```

### **Playbook using vars file**

```yaml
---
- name: Web configuration using vars file
  hosts: webservers
  vars_files:
    - web_vars.yml

  tasks:
    - name: Install package
      package:
        name: "{{ web_package }}"
        state: present

    - name: Create document root
      file:
        path: "{{ web_root }}"
        state: directory
```

---

# **3.2.3 Inventory Variables**

Variables can also be placed directly in the inventory.

### **Example inventory.ini**

```ini
[webservers]
web1 ansible_host=192.168.10.10 web_port=8080
web2 ansible_host=192.168.10.11 web_port=9090
```

### Use the variable in a playbook:

```yaml
- name: Display port for each server
  hosts: webservers
  tasks:
    - debug:
        msg: "Server {{ inventory_hostname }} uses port {{ web_port }}"
```

---

# **3.2.4 Group Variables (group_vars)**

Directory structure:

```
group_vars/
  webservers.yml
  databases.yml
```

### **Example: group_vars/webservers.yml**

```yaml
web_package: nginx
```

---

# **3.2.5 Host Variables (host_vars)**

Directory structure:

```
host_vars/
  web1.yml
  web2.yml
```

### **Example: host_vars/web1.yml**

```yaml
web_port: 8080
```

---

# **3.2.6 Extra Variables (`-e`)**

Used to override everything from the command line (highest precedence).

```bash
ansible-playbook deploy.yml -e "version=1.0.1"
```

Use inside playbook:

```yaml
debug:
  msg: "Deploying version {{ version }}"
```

---

# **3.3 Variable Precedence (Simple Explanation)**

Ansible has many levels of precedence, but beginners only need to remember:

**Highest → Lowest**

1. Extra vars `-e`
2. Task vars
3. Play vars
4. Vars_files
5. Inventory vars
6. group_vars / host_vars
7. Role defaults (lowest)

This determines which value is used if a variable is defined in multiple places.

---

# **3.4 Facts in Ansible**

Facts are automatically collected information about managed hosts, such as:

* `ansible_hostname`
* `ansible_distribution` (Ubuntu, CentOS, etc.)
* `ansible_default_ipv4.address`
* `ansible_memory_mb`
* `ansible_nodename`
* `ansible_processor`

Ansible gathers facts at the start of each play **unless disabled**.

### **Example: Print all facts**

```bash
ansible all -m setup
```

This displays a long JSON output of host facts.

---

# **3.4.1 Using Facts Inside Playbooks**

### **Example Playbook**

```yaml
---
- name: Using facts
  hosts: all

  tasks:
    - name: Print OS type
      debug:
        msg: "This server is running {{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: Print server IP
      debug:
        msg: "Primary IP is {{ ansible_default_ipv4.address }}"
```

---

# **3.4.2 Disabling Fact Gathering (Optional)**

If you want faster execution and don’t need system information:

```yaml
gather_facts: no
```

---

# **3.4.3 Using Facts in Templates**

Example Jinja2 template (`system_info.j2`):

```
Hostname: {{ ansible_hostname }}
Operating System: {{ ansible_distribution }} {{ ansible_distribution_version }}
IP Address: {{ ansible_default_ipv4.address }}
```

Playbook to apply template:

```yaml
- name: Generate system report
  hosts: all

  tasks:
    - template:
        src: system_info.j2
        dest: /tmp/system_report.txt
```
Below is the next section in the same **README.md**–ready format.

---

# **4. Control Flow**

Control flow in Ansible allows you to add logic to your playbooks.
It helps you control:

✔ When a task should run
✔ How many times a task should run
✔ What happens when something fails
✔ How to run only specific tasks using tags

This section explains **conditionals**, **loops**, and **tags** with beginner-friendly examples.

---

# **4.1 Conditionals (when)**

Conditionals allow tasks to run only when certain conditions are met.

### **Why Conditionals Are Useful**

* Apply tasks only on specific OS types (Ubuntu, RedHat, etc.)
* Install different packages on different systems
* Run tasks based on variable values
* Avoid unnecessary or dangerous tasks

---

## **Basic Example of `when`**

```yaml
- name: Install Apache on Ubuntu
  apt:
    name: apache2
    state: present
  when: ansible_distribution == "Ubuntu"
```

If the system is not Ubuntu, the task is skipped.

---

## **Multiple Conditions**

You can combine conditions using `and` / `or`.

```yaml
when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
```

---

## **Condition Using Variables**

```yaml
vars:
  deploy_app: true

tasks:
  - name: Deploy application
    debug:
      msg: "Deploying application!"
    when: deploy_app
```

---

# **4.1.1 changed_when & failed_when**

Sometimes Ansible incorrectly marks tasks as changed or failed.
You can override this behavior.

---

### **Example: Mark task as changed only when output contains 'updated'**

```yaml
- name: Custom change detection
  command: /usr/bin/update_app
  register: result
  changed_when: "'updated' in result.stdout"
```

---

### **Example: Prevent a command from failing playbook**

```yaml
- name: Check service status safely
  command: systemctl status myapp
  register: output
  failed_when: false
```

---

# **4.2 Loops**

Loops let you repeat the same task for multiple values.

---

## **Basic Loop Example**

```yaml
- name: Create multiple users
  user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - chris
```

---

## **Looping Through Dictionaries**

```yaml
- name: Create users with UID
  user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
  loop:
    - { name: "alice", uid: 2001 }
    - { name: "bob", uid: 2002 }
```

---

## **Looping with with_items (older syntax)**

```yaml
with_items:
  - nginx
  - git
  - curl
```

Modern versions recommend using `loop:` instead.

---

# **4.3 Tags**

### Tags: Running or Skipping Specific Tasks

Tags allow you to organize and selectively execute tasks, roles, or blocks in a playbook. This is useful when you want to run only certain parts of a playbook without executing everything.

#### Example: Defining Tags in a Playbook

```yaml
---
- name: Deploy Web Application
  hosts: webservers
  become: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
      tags:
        - install
        - webserver

    - name: Start Nginx service
      service:
        name: nginx
        state: started
      tags:
        - start
        - webserver

    - name: Configure firewall
      ufw:
        rule: allow
        port: 80
        proto: tcp
      tags:
        - firewall
```

#### Running Only Tasks with a Specific Tag

```bash
ansible-playbook deploy_web.yaml --tags "install"
```

This command will only execute tasks tagged with `install`.

#### Skipping Tasks with a Specific Tag

```bash
ansible-playbook deploy_web.yaml --skip-tags "firewall"
```

This command will run the playbook but skip all tasks tagged with `firewall`.

#### Benefits of Using Tags

* Run only a subset of tasks during testing or maintenance.
* Save time by skipping tasks that don’t need to run.
* Organize large playbooks into manageable sections.

---


# **5. Handlers and Notifications**

Handlers are special tasks in Ansible that run **only when notified** by other tasks.
They are typically used for actions that should happen **only after a change occurs**, such as:

✔ Restarting a service
✔ Reloading a configuration
✔ Restarting a database
✔ Triggering dependent processes

This prevents unnecessary restarts and improves idempotency.

---

# **5.1 What Are Handlers?**

A **handler** is a task that runs at the *end* of a play if it has been triggered.

Handlers are defined using:

```yaml
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

---

# **5.2 What Are Notifications?**

A notification tells a handler to run.
It is used when a task **changes something**.

Example:

```yaml
notify: restart nginx
```

The handler will run **only if the task reports “changed”**.

---

# **5.3 Why Use Handlers? (Beginner Explanation)**

Imagine you copy a new configuration file:

* If the file was actually updated → Restart the service
* If nothing changed → Do NOT restart

Handlers make this automatic and smart.

This is ideal for:

✔ Web server configuration
✔ Database updates
✔ Firewall changes
✔ Application deployments

---

# **5.4 Complete Example: Restarting Service After Config Change**

```yaml
---
- name: Configure nginx with handlers
  hosts: webservers
  become: yes

  tasks:
    - name: Copy nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: restart nginx  # Trigger handler only if file changed

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### **Explanation**

1. If `nginx.conf` changes → handler runs
2. If file is identical → handler does *not* run
3. Handlers run at the **end of the play**, not immediately

---

# **5.5 Multiple Notifications**

A task can notify multiple handlers.

```yaml
notify:
  - restart nginx
  - reload firewall
```

---

# **5.6 Multiple Tasks Notifying Same Handler**

This is common.

```yaml
- name: Install nginx
  package:
    name: nginx
    state: present
  notify: restart nginx

- name: Update nginx config
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

Handler runs **only once**, even if multiple tasks notify it.

---

# **5.7 “listen” Feature (Advanced but Beginner-Friendly)**

Allows multiple handler names to trigger the same action.

### Example:

```yaml
handlers:
  - name: restart web service
    listen: restart_nginx
    service:
      name: nginx
      state: restarted

  - name: reload firewall
    listen: reload_fw
    service:
      name: firewalld
      state: reloaded
```

Tasks can notify:

```yaml
notify: restart_nginx
```
Here’s the next section, fully formatted for **README.md** with beginner-friendly explanations and examples.

---

# **6. Files and Templates**

Ansible makes managing files and configurations easy using the **copy** and **template** modules.
Templates allow you to create **dynamic configuration files** using **Jinja2**.

---

# **6.1 Copy Module**

The **copy** module copies files from the **control node** to **managed nodes**.

### Example: Copy a File

```yaml
---
- name: Copy configuration file
  hosts: all
  become: yes

  tasks:
    - name: Copy nginx.conf to server
      copy:
        src: files/nginx.conf   # File on control node
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
```

**Explanation:**

* `src`: Source file on the control node
* `dest`: Destination path on managed node
* `owner`, `group`, `mode`: File permissions

**Benefits:**

* Ensures consistent configuration files across multiple servers
* Easy to update configurations centrally

---

# **6.2 File Module**

The **file** module manages **files and directories**.

### Example: Create a Directory

```yaml
- name: Create a logs directory
  file:
    path: /var/log/myapp
    state: directory
    owner: ansible
    group: ansible
    mode: '0755'
```

**Other Uses:**

* `state: absent` → delete a file/directory
* `state: touch` → create an empty file

---

# **6.3 Template Module**

Templates are **dynamic files** with variables and logic.
They use **Jinja2 syntax** (`{{ variable_name }}`).

### Example: Nginx Template

`templates/nginx.conf.j2`:

```jinja
server {
    listen 80;
    server_name {{ server_name }};
    root {{ document_root }};
}
```

Playbook:

```yaml
- name: Deploy nginx config template
  hosts: webservers
  become: yes

  vars:
    server_name: example.com
    document_root: /var/www/html

  tasks:
    - name: Apply nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
```

**Explanation:**

* `{{ server_name }}` and `{{ document_root }}` are replaced with variable values
* Templates make it easy to **reuse configs across multiple servers**

---

# **6.4 Common Jinja2 Filters**

Jinja2 allows **modifying variables** inside templates:

| Filter    | Example           | Output               |           |
| --------- | ----------------- | -------------------- | --------- |
| `upper`   | `{{ 'ansible'     | upper }}`            | `ANSIBLE` |
| `lower`   | `{{ 'ANSIBLE'     | lower }}`            | `ansible` |
| `default` | `{{ undefined_var | default('value') }}` | `value`   |
| `replace` | `{{ 'foo'         | replace('f','b') }}` | `boo`     |

**Example in Template:**

```jinja
server_name {{ server_name | lower }};
```

---

# **6.5 Why Use Files and Templates?**

* Centralized management of **configuration files**
* Dynamic templates adapt to **different servers/environments**
* Reduces errors and ensures **idempotency**
* Supports automation at scale
---

Here’s the next section in **README.md** format, fully explained for beginners:

---

# **7. Execution and Debugging**

Ansible provides powerful tools to **execute playbooks**, **control execution**, and **debug issues** efficiently.

---

# **7.1 Ansible CLI**

Ansible has two main commands:

1. **`ansible`** – Run **ad-hoc commands** on managed nodes.
2. **`ansible-playbook`** – Execute **playbooks** for complex tasks and automation.

### Example: Ping All Hosts

```bash
ansible all -m ping -i inventory.ini
```

* `all` → target all hosts in the inventory
* `-m ping` → use the ping module to check connectivity
* `-i inventory.ini` → specify the inventory file

**Output:**

```
server1 | SUCCESS => {"changed": false, "ping": "pong"}
server2 | SUCCESS => {"changed": false, "ping": "pong"}
```

---

# **7.2 Playbook Execution Options**

### Limiting Hosts

```bash
ansible-playbook deploy_app.yml -l webservers
```

* `-l webservers` → run only on the `webservers` group

### Check Mode (Dry Run)

```bash
ansible-playbook deploy_app.yml --check
```

* Shows what **changes would be made** without applying them

### Verbosity

```bash
ansible-playbook deploy_app.yml -v
ansible-playbook deploy_app.yml -vvv
```

* `-v` → basic debug info
* `-vvv` → detailed debug info for troubleshooting

---

# **7.3 Debug Module**

The `debug` module helps **inspect variables and outputs**.

### Example: Debug a Variable

```yaml
- name: Debug example
  hosts: all
  vars:
    app_port: 8080
  tasks:
    - name: Show app port
      debug:
        msg: "Application will run on port {{ app_port }}"
```

**Output:**

```
TASK [Show app port] ****************************************************************
ok: [server1] => {
    "msg": "Application will run on port 8080"
}
```

---

# **7.4 Register Variables**

`register` stores the **output of a task** in a variable for later use.

### Example: Check Service Status

```yaml
- name: Check nginx status
  service:
    name: nginx
    state: started
  register: nginx_status

- name: Debug nginx status
  debug:
    var: nginx_status
```

**Benefits:**

* Helps in **conditional tasks**
* Makes playbooks **dynamic and reusable**

---

# **7.5 Why Execution and Debugging Matters**

* Ensure **playbooks run correctly** before deploying changes
* Debug issues **quickly and safely**
* Run **safe dry-runs** to avoid breaking production systems
* Provides **visibility into automation results**

---

# **8. Security and Secrets**

Automation often requires handling **sensitive data** like passwords, API keys, and certificates. Ansible provides a secure way to manage secrets through **Ansible Vault**.

---

# **8.1 What is Ansible Vault?**

Ansible Vault allows you to:

* **Encrypt sensitive files** (YAML, vars files, playbooks)
* **Decrypt them only when needed**
* **Keep secrets out of version control**

It ensures that passwords and keys are **never exposed in plain text**.

---

# **8.2 Creating a Vault File**

```bash
ansible-vault create secrets.yml
```

* Opens your default text editor
* You can add sensitive variables:

```yaml
db_user: admin
db_password: StrongP@ssw0rd
```

* Save and exit → file is now encrypted

---

# **8.3 Editing an Encrypted File**

```bash
ansible-vault edit secrets.yml
```

* Opens the encrypted file for modification
* After saving, it remains encrypted

---

# **8.4 Encrypting Existing Files**

```bash
ansible-vault encrypt vars.yml
```

* Encrypts an existing file in place

---

# **8.5 Decrypting Files**

```bash
ansible-vault decrypt secrets.yml
```

* Makes the file readable again (use with caution)

---

# **8.6 Using Vault in Playbooks**

You can reference encrypted variables in your playbooks just like normal variables:

```yaml
- name: Deploy application with secret
  hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Show DB user
      debug:
        msg: "Database user is {{ db_user }}"
```

Run the playbook by providing the vault password:

```bash
ansible-playbook deploy_app.yml --ask-vault-pass
```

---

# **8.7 Best Practices for Secrets**

1. **Do not hardcode secrets** in playbooks
2. **Use separate vault files** for sensitive information
3. **Keep vault passwords secure** and use a password manager if possible
4. **Consider environment-specific vaults** (e.g., dev, prod)

---

# **8.8 Benefits of Using Vault**

* Keeps automation **secure**
* **Centralizes secret management**
* Allows **safe collaboration** among team members
* Integrates smoothly with **CI/CD pipelines**

---

# **9. Best Practices (Intro)**

Following best practices in Ansible helps **maintain clean, reusable, and reliable automation code**. It ensures your playbooks are **readable, maintainable, and scalable** for teams and production environments.

---

# **9.1 Directory Layout**

A well-structured directory layout makes it easier to manage playbooks, roles, and variables. Recommended structure:

```
ansible-project/
├── inventories/
│   ├── dev/
│   └── prod/
├── roles/
│   ├── webserver/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   ├── files/
│   │   ├── vars/
│   │   ├── defaults/
│   │   └── meta/
├── playbooks/
│   ├── site.yml
│   └── deploy_app.yml
├── group_vars/
├── host_vars/
└── ansible.cfg
```

* **inventories/**: Contains environment-specific hosts
* **roles/**: Reusable components
* **playbooks/**: High-level workflows
* **group_vars/** & **host_vars/**: Store variables per group or host
* **ansible.cfg**: Configuration file for the project

---

# **9.2 Using Roles for Reuse**

Roles allow **encapsulating tasks, handlers, templates, and variables** into reusable units.

**Benefits of roles:**

* Easier collaboration
* Reduced code duplication
* Simplifies maintenance and testing

Example usage:

```yaml
- name: Configure webserver
  hosts: webservers
  roles:
    - webserver
```

---

# **9.3 Keeping Playbooks Idempotent**

Idempotence ensures that running a playbook **multiple times does not change the system if it's already in the desired state**.

* Avoid shell commands unless necessary
* Use Ansible modules (package, service, file, user, etc.)
* Always define `state` for resources

Example:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

Running this playbook multiple times will **not reinstall nginx** if it is already installed.

---

# **9.4 Naming Conventions and Comments**

* Use **descriptive task names**:

```yaml
- name: Ensure Apache service is running
  service:
    name: apache2
    state: started
```

* Add comments for clarity:

```yaml
# Create the web root directory if it doesn't exist
- name: Ensure /var/www/html exists
  file:
    path: /var/www/html
    state: directory
```

---

# **9.5 Using ansible-lint (Intro)**

`ansible-lint` is a tool that **checks your playbooks for best practices** and potential errors.

Install:

```bash
pip install ansible-lint
```

Run:

```bash
ansible-lint playbook.yml
```

Benefits:

* Enforces coding standards
* Detects common mistakes
* Improves readability and maintainability

---


# **10. Inventories (Advanced)**

Ansible inventories define **which hosts you manage** and how they are organized. Advanced inventory usage improves **flexibility and environment management**.

---

# **10.1 Group_vars and Host_vars**

* **group_vars/**: Variables applied to a **group of hosts**
* **host_vars/**: Variables applied to a **specific host**

**Example directory structure:**

```
inventories/
├── dev/
│   ├── hosts
│   ├── group_vars/
│   │   └── webservers.yml
│   └── host_vars/
│       └── server1.yml
```

**group_vars/webservers.yml**

```yaml
nginx_port: 80
document_root: /var/www/html
```

**host_vars/server1.yml**

```yaml
ansible_user: deployer
ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

* Playbooks automatically pick these variables based on **host membership**.

---

# **10.2 Dynamic Inventory**

Dynamic inventories allow you to **fetch hosts from external sources**, such as cloud providers or scripts.

**Why use dynamic inventory?**

* Automatic scaling of hosts
* No manual inventory updates for cloud resources
* Ideal for CI/CD pipelines and cloud automation

**Example: Using AWS EC2 Plugin**

```yaml
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: dev
keyed_groups:
  - key: tags.Role
    prefix: role
```

* This inventory automatically lists EC2 instances with `Environment=dev`
* Hosts are grouped by their `Role` tag

**Manual Script Example**

* You can write a Python script to output JSON for Ansible inventory:

```python
#!/usr/bin/env python3
import json

inventory = {
    "webservers": {
        "hosts": ["192.168.1.10", "192.168.1.11"],
        "vars": {"nginx_port": 80}
    }
}
print(json.dumps(inventory))
```

Run the playbook with dynamic inventory:

```bash
ansible-playbook -i my_inventory_script.py deploy.yml
```

---


# **11. Roles in Depth**

Roles in Ansible allow you to **organize playbooks into reusable, modular components**. They are useful for structuring complex automation and promoting **consistency and maintainability**.

---

# **11.1 Standard Role Structure**

A typical role has the following structure:

```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml       # Core tasks of the role
    ├── handlers/
    │   └── main.yml       # Handlers for notifications (e.g., restart services)
    ├── templates/         # Jinja2 templates for configuration files
    │   └── nginx.conf.j2
    ├── files/             # Static files to copy
    │   └── index.html
    ├── vars/
    │   └── main.yml       # Role-specific variables
    ├── defaults/
    │   └── main.yml       # Default variables (lowest precedence)
    └── meta/
        └── main.yml       # Role metadata and dependencies
```

**Key Points:**

* **tasks/main.yml** – Defines all the tasks to configure the service or application
* **handlers/main.yml** – Handles notifications like service restarts
* **templates/** – Contains Jinja2 templates for dynamic configuration files
* **defaults/main.yml** – Default values for variables (lowest priority)
* **vars/main.yml** – Variables with higher priority than defaults
* **meta/main.yml** – Optional metadata such as role dependencies

---

# **11.2 Using Roles in a Playbook**

Example: Applying a `webserver` role to `webservers` hosts:

```yaml
---
- name: Deploy webserver role
  hosts: webservers
  become: yes

  roles:
    - webserver
```

* Ansible automatically looks for the `webserver` role in `roles/`
* Executes tasks, handlers, templates, and files as defined in the role

---

# **11.3 Role Dependencies**

Roles can depend on other roles, specified in **meta/main.yml**:

```yaml
dependencies:
  - role: common
  - role: firewall
```

* When the `webserver` role runs, it **automatically runs dependent roles first**
* Helps modularize configurations (e.g., common users, firewall setup)

---

# **11.4 Benefits of Using Roles**

* Encourages **reuse** of automation code
* Makes playbooks **cleaner and easier to maintain**
* Allows **standardization** across multiple environments
* Supports **role testing** independently before integration

---


# **12. Advanced Variables and Facts**

Advanced variable management and facts in Ansible allow for **dynamic, environment-specific automation**. Understanding precedence and how to manipulate facts is crucial for robust playbooks.

---

# **12.1 Variable Precedence**

Ansible variables can come from multiple sources, and **precedence determines which value is used**. Higher-precedence variables override lower-precedence ones.

| Precedence Level | Source                                                  |
| ---------------- | ------------------------------------------------------- |
| 1                | Extra vars (`ansible-playbook play.yml -e "var=value"`) |
| 2                | Task vars (`vars:` within a task)                       |
| 3                | Block vars                                              |
| 4                | Role vars (`vars/main.yml`)                             |
| 5                | Play vars (`vars:` in playbook)                         |
| 6                | Host vars (`host_vars/`)                                |
| 7                | Group vars (`group_vars/`)                              |
| 8                | Defaults (`defaults/main.yml` in roles)                 |
| 9                | Facts (`gather_facts`)                                  |
| 10               | Inventory variables                                     |

**Example: Overriding a Variable**

```yaml
# defaults/main.yml in a role
nginx_port: 80

# vars/main.yml in the same role
nginx_port: 8080

# Playbook level
- hosts: webservers
  vars:
    nginx_port: 9090
```

* Final value of `nginx_port` = **9090** (playbook vars override role vars and defaults)

---

# **12.2 Using `set_fact`**

`set_fact` allows you to **define variables dynamically** during playbook execution.

**Example:**

```yaml
- name: Set a dynamic variable
  hosts: all
  tasks:
    - name: Calculate port dynamically
      set_fact:
        dynamic_port: "{{ 8000 + inventory_hostname | regex_replace('[^0-9]', '') | int }}"

    - debug:
        msg: "Server {{ inventory_hostname }} will use port {{ dynamic_port }}"
```

* Useful when values depend on hostnames, calculations, or other facts

---

# **12.3 Facts and Gathering System Information**

Ansible can gather system facts using the `setup` module:

```yaml
- name: Gather facts
  hosts: all
  tasks:
    - name: Show OS info
      debug:
        msg: "OS: {{ ansible_facts['os_family'] }}, Version: {{ ansible_facts['distribution_version'] }}"
```

* Facts provide details like **IP addresses, OS type, memory, CPU**
* Can be **cached** for efficiency using `fact_caching` in `ansible.cfg`:

```ini
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600
```

---

# **12.4 Benefits of Advanced Variables and Facts**

* Enables **dynamic configurations** for different hosts
* Reduces **hardcoding** in playbooks
* Makes playbooks **more flexible and reusable**
* Helps in **conditional execution** and complex deployments

---



# **13. Blocks and Error Handling**

In Ansible, **blocks** help you group tasks together and handle errors gracefully. They are especially useful in complex automation workflows where some tasks might fail, but you want to continue with others or perform cleanup actions.

---

# **13.1 Using `block`, `rescue`, and `always`**

* **`block`**: Groups tasks together.
* **`rescue`**: Runs only if tasks in the block fail.
* **`always`**: Runs regardless of success or failure, similar to a `finally` clause in programming.

---

# **13.2 Example: Handling Service Deployment Failure**

```yaml
- name: Deploy application with error handling
  hosts: webservers
  become: yes

  tasks:
    - name: Block example
      block:
        - name: Stop nginx service
          service:
            name: nginx
            state: stopped

        - name: Deploy configuration file
          copy:
            src: /tmp/nginx.conf
            dest: /etc/nginx/nginx.conf

        - name: Start nginx service
          service:
            name: nginx
            state: started

      rescue:
        - name: Rollback configuration on failure
          copy:
            src: /tmp/nginx.conf.bak
            dest: /etc/nginx/nginx.conf

        - name: Ensure nginx service is running
          service:
            name: nginx
            state: started

      always:
        - name: Clean temporary files
          file:
            path: /tmp/nginx.conf
            state: absent
```

**Explanation:**

* `block`: Attempts to stop nginx, deploy new config, and start service.
* `rescue`: If any task in the block fails, rolls back configuration and restarts nginx.
* `always`: Ensures temporary files are removed whether the tasks succeed or fail.

---

# **13.3 Benefits of Using Blocks**

* Improves **playbook reliability**
* Makes playbooks **more readable and maintainable**
* Prevents **partial configuration states** in case of failure
* Allows **graceful recovery** and cleanup actions

---


# **14. Advanced Templating**

Ansible uses **Jinja2 templating** to make configuration files dynamic. Advanced templating allows you to use **conditionals, loops, and lookups** inside templates, making them flexible and reusable.

---

# **14.1 Conditionals in Templates**

You can use `if` statements inside templates to render content based on variable values.

**Example: nginx.conf.j2**

```jinja
server {
    listen 80;

    server_name {{ server_name }};

    {% if enable_ssl %}
    listen 443 ssl;
    ssl_certificate {{ ssl_cert }};
    ssl_certificate_key {{ ssl_key }};
    {% endif %}

    root {{ document_root }};
}
```

**Explanation:**

* `{{ server_name }}` and `{{ document_root }}` are variables replaced with actual values from the playbook.
* The `{% if enable_ssl %}` block is included only if `enable_ssl` is `true`.

---

# **14.2 Loops in Templates**

You can iterate over lists or dictionaries using `for` loops.

**Example: users.j2**

```jinja
{% for user in users %}
useradd {{ user.name }}
{% if user.sudo %}
usermod -aG sudo {{ user.name }}
{% endif %}
{% endfor %}
```

**Explanation:**

* Iterates over a list of `users`
* Creates a user and optionally adds them to `sudo` group based on a property

**Playbook Example:**

```yaml
- name: Create multiple users
  hosts: all
  vars:
    users:
      - { name: alice, sudo: true }
      - { name: bob, sudo: false }

  tasks:
    - name: Deploy user creation script
      template:
        src: users.j2
        dest: /tmp/create_users.sh
      mode: '0755'
```

---

# **14.3 Lookup Plugins in Templates**

**Lookup plugins** fetch external data dynamically.

Common examples: `file`, `env`, `password`.

**Example: Load secret password from a file**

```jinja
db_password = {{ lookup('file', '/tmp/db_pass.txt') }}
```

**Example: Fetch environment variable**

```jinja
path_variable = {{ lookup('env', 'PATH') }}
```

**Example: Generate password**

```jinja
admin_password = {{ lookup('password', '/tmp/admin_pass.txt length=16 chars=ascii_letters') }}
```

**Benefits of Advanced Templating:**

* Reduces repetitive playbook logic
* Makes templates **dynamic and reusable**
* Centralizes configuration management
* Integrates external data securely and efficiently

---

# **15.Ansible Galaxy**

Ansible Galaxy is a **hub for pre-built Ansible roles and collections** created by the community or Red Hat. It allows you to **reuse automation code**, saving time and ensuring best practices without writing everything from scratch.

Perfect! Here’s a polished, GitHub-ready combined section for **Ansible Galaxy and Collections**, ready to paste into your README.md:

---

## Ansible Galaxy and Collections

Ansible Galaxy and Collections are tools to **reuse automation code** created by the community or vendors. They allow you to save time, follow best practices, and extend Ansible’s capabilities without writing everything from scratch.

---

### Ansible Galaxy

**Ansible Galaxy** is a hub for pre-built **roles**.

**Benefits of Using Galaxy:**

* Save time with tested roles for common tasks (web servers, databases, monitoring, etc.)
* Follow community best practices
* Access extra modules and plugins

**Installing a Role from Galaxy:**

```bash
ansible-galaxy install geerlingguy.nginx
```

**Using a Role in a Playbook:**

```yaml
---
- name: Setup Nginx using Galaxy role
  hosts: webservers
  become: yes
  roles:
    - geerlingguy.nginx
```

**Searching for Roles:**

```bash
ansible-galaxy search nginx
```

**Best Practices:**

* Check role documentation for required variables and compatibility
* Use roles with active maintenance and good ratings
* Pin roles to specific versions to avoid breaking changes

---

### Ansible Collections
**Ansible Galaxy is a hub for pre-built Ansible roles and collections created by the community or Red Hat. It allows you to reuse automation code, saving time and ensuring best practices without writing everything from scratch.**
**Ansible Collections** are packaged units of **roles, modules, and plugins**.

**Benefits of Using Collections:**

* Modularity: Bundle multiple resources for specific tasks
* Reusability: Share across multiple projects
* Extended functionality: Access modules and plugins not in core Ansible

**Installing a Collection:**

```bash
ansible-galaxy collection install community.general
```

**Example: Creating an AWS EC2 Instance using a Collection**

```yaml
---
- name: Launch EC2 instance using community.general
  hosts: localhost
  gather_facts: no
  vars:
    aws_region: us-east-1
    ec2_key: my-key
    ec2_instance_type: t2.micro
    ec2_image: ami-0c94855ba95c71c99  # Amazon Linux 2 AMI
  tasks:
    - name: Launch EC2 instance
      community.general.ec2_instance:
        name: my-test-instance
        key_name: "{{ ec2_key }}"
        instance_type: "{{ ec2_instance_type }}"
        image_id: "{{ ec2_image }}"
        region: "{{ aws_region }}"
        wait: yes
        count: 1
        state: present
      register: ec2
```

**Explanation:**

* `community.general.ec2_instance`: Module from the collection for managing EC2 instances
* `register: ec2`: Stores output for later use
* `wait: yes`: Waits until the instance is ready

**Best Practices:**

* Pin collection versions to avoid breaking changes
* Review module documentation for required parameters
* Use variables for playbook reusability across environments

---

