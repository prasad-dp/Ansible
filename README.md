# Ansible: Your Automation Assistant - Quick Revision Note

[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ansible](https://img.shields.io/badge/Ansible-%23EE0000.svg?style=flat&logo=ansible&logoColor=white)](https://www.ansible.com/)

This document provides a quick revision of Ansible, a powerful open-source automation tool. It covers core concepts, practical procedures, and essential organizational structures.

## What is Ansible?

Ansible is an open-source automation engine that simplifies IT tasks such as configuration management, application deployment, task automation, and orchestration. It allows you to manage and configure a large number of systems efficiently and consistently from a central location. Ansible is known for its human-readable language (YAML), its agentless architecture (relying on SSH or WinRM), and its powerful capabilities.

**Use Cases of Ansible:**

Ansible is versatile and can be used for a wide range of IT automation tasks, including:

* **Configuration Management:** Ensuring systems are configured in a desired state.
* **Application Deployment:** Deploying applications to multiple servers simultaneously.
* **Orchestration:** Coordinating complex workflows across different systems.
* **Provisioning:** Setting up new infrastructure (servers, cloud resources, etc.).
* **Continuous Delivery:** Automating the software release process.
* **Security Automation:** Automating security tasks like patching and user management.

## Core Components

* **Control Node (Management Node):** The machine where Ansible is installed and from which you run your automation playbooks. It's the central point of control.

* **Managed Nodes (Targets):** The servers, network devices, or other systems that Ansible manages, typically via SSH or WinRM.

* **Inventory File:** A file (INI or YAML) listing managed nodes, organized into groups, allowing targeted execution of playbooks.
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

## Installation Guide

Instructions to install Ansible on your control node.

* **Prerequisites:** Python 3 and `pip` (recommended).

* **Installation Methods:**

    * **Using `pip` (Recommended):**
        ```bash
        pip install ansible
        ```

    * **Debian/Ubuntu:**
        ```bash
        sudo apt update && sudo apt install software-properties-common -y
        sudo add-apt-repository --yes --update ppa:ansible/ansible
        sudo apt install ansible -y
        ```

    * **Red Hat/CentOS/Fedora:**
        ```bash
        sudo dnf install epel-release -y  # For Fedora, CentOS 8 or later
        sudo yum install epel-release -y  # For CentOS 7
        sudo dnf install ansible -y       # For Fedora, CentOS 8 or later
        sudo yum install ansible -y       # For CentOS 7
        ```

    * **macOS:**
        ```bash
        brew install ansible
        ```

* **Verify Installation:**
    ```bash
    ansible --version
    ```

# Ansible System Modules: Core Management Tools

Ansible's power comes from its **modules**, which are pre-built tools designed to automate specific tasks on your managed systems. They abstract away the underlying commands needed for different operating systems, providing a consistent way to manage your infrastructure.

## What are Ansible Modules?

Ansible modules are self-contained pieces of code that Ansible executes on managed nodes. They take parameters that define the desired state or action and handle the complexities of interacting with the target system to achieve that state. Modules are designed to be idempotent, meaning running them multiple times will always result in the same system state.

## Use of Modules

Modules are the fundamental building blocks of Ansible playbooks. They allow you to:

* **Automate repetitive tasks:** Configure systems, install software, manage services consistently across many machines.
* **Ensure desired state:** Define the exact configuration you want for your systems, and Ansible will ensure they are in that state.
* **Work across different operating systems:** Modules handle the differences between Linux distributions (like Debian, Red Hat, etc.) and other systems.
* **Simplify automation:** You don't need to write complex shell scripts; modules provide a higher-level, more structured way to manage systems.

## Individual System Modules and Examples

Here are the system modules you listed with explanations and examples of their use in a playbook:

```yaml
---
- name: Examples of Core System Modules
  hosts: all
  become: yes  # For tasks requiring root privileges

  tasks:
    - name: Use the 'yum' module (for Red Hat/CentOS/Fedora)
      yum:
        name: httpd
        state: present
      # Ensures the 'httpd' (Apache) package is installed.

    - name: Use the 'apt' module (for Debian/Ubuntu)
      apt:
        name: nginx
        state: latest
      # Ensures the 'nginx' package is installed and is the latest available version.

    - name: Use the 'dnf' module (for Fedora/CentOS 8+)
      dnf:
        name: mariadb-server
        state: absent
      # Ensures the 'mariadb-server' package is removed.

    - name: Use the 'package' module (generic package management)
      package:
        name: curl
        state: installed
      # A more generic module that tries to use the appropriate package manager
      # for the target system (e.g., apt, yum, dnf). 'installed' is similar to 'present'.

    - name: Use the 'service' module (manage system services)
      service:
        name: firewalld
        state: stopped
        enabled: no
      # Stops the 'firewalld' service and disables it from starting on boot.
      # Other common states include 'started', 'restarted', 'reloaded'.

    - name: Use the 'copy' module (copy files to managed nodes)
      copy:
        src: /tmp/local_config.txt
        dest: /etc/remote_config.txt
        owner: root
        group: root
        mode: '0644'
      # Copies the file '/tmp/local_config.txt' from the control node
      # to '/etc/remote_config.txt' on the managed node, setting ownership and permissions.

    - name: Use the 'file' module (manage files and directories)
      file:
        path: /opt/new_directory
        state: directory
        owner: ansible
        group: ansible
        mode: '0755'
      # Creates the directory '/opt/new_directory' if it doesn't exist,
      # setting the owner, group, and permissions.
      # Other states include 'file' (creates an empty file), 'absent' (removes), 'touch'.

    - name: Use the 'ping' module (check host reachability)
      ping:
      # Sends a test ping to the managed nodes to verify they are reachab

# Ansible Variables in YAML: A Concise Guide

Variables are key to making your Ansible playbooks dynamic and reusable. This guide demonstrates how to define and use variables within Ansible using YAML syntax, both directly in playbooks and in separate files.

## Defining Variables Directly in a Playbook (YAML)

For simple, playbook-specific configurations, you can define variables right within the `vars` section of your playbook.

```yaml
---
- name: Simple variable example in a playbook
  hosts: all
  vars:
    server_name: "webserver-prod-01"
    http_port: 80

  tasks:
    - name: Output the server name
      debug:
        msg: "Configuring server: {{ server_name }}"

    - name: Set the HTTP port
      debug:
        msg: "HTTP port will be: {{ http_port }}"

To run this playbook (e.g., server_config.yaml), execute:
ansible-playbook server_config.yaml

Storing Variables in Separate YAML Files (Vars Files)
For better organization, especially when you have variables that might be used across multiple playbooks, you can store them in external YAML files (like web_packages.yaml).
Example of a Vars File (web_packages.yaml):
apache_package: httpd

To use this external variable, reference the vars file in your playbook using the vars_files directive:
---
- name: Example using an external vars file
  hosts: webservers
  vars_files:
    - web_packages.yaml

  tasks:
    - name: Ensure Apache is installed
      package:
        name: "{{ apache_package }}"
        state: present

To run this playbook (e.g., install_apache.yaml) that uses web_packages.yaml, simply run:
ansible-playbook install_apache.yaml

Ansible will automatically load the variable apache_package from the specified file. You can also explicitly specify the vars file using the --vars-file option:
ansible-playbook install_apache.yaml --vars-file web_packages.yaml


Roles

        

