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

## Modules

Pre-built tools that perform specific tasks on managed nodes (e.g., installing packages, managing services).

**Example:** Ensure Nginx is installed on web servers.

```yaml
- name: Ensure nginx is installed
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx package
      apt:
        name: nginx
        state: present

Variables
Placeholders for dynamic values, making automation flexible. Defined in inventory, playbooks, or variable files. Accessed using Jinja2 ({{ variable_name }}).
Example: Define and use a variable for the HTTP port.
 * Inventory (hosts):
   [webservers]
server1.example.com http_port=80
server2.example.com http_port=81

 * Playbook:
   - name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure nginx is listening on the correct port
      lineinfile:
        path: /etc/nginx/sites-available/default
        regexp: '^listen '
        line: "listen {{ http_port }} default_server;"
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted

Roles
A structured way to organize and reuse Ansible content (tasks, handlers, variables, templates).
Example: A basic role structure for setting up a web server.
roles/
└── webserver_setup/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    └── templates/
        └── index.html.j2

Using the role in a playbook:
---
- name: Deploy web server
  hosts: webservers
  become: yes
  roles:
    - webserver_setup

Basic Example: Hosting an HTML File on Nginx Port 80
A simple playbook to install Nginx and host an index.html file on port 80.
 * Inventory (hosts):
   [webservers]
your_server_ip_or_hostname

 * index.html:
   <!DOCTYPE html>
<html>
<head>
    <title>My Ansible Hosted Page</title>
</head>
<body>
    <h1>This page is served by Nginx, configured by Ansible!</h1>
</body>
</html>

 * Playbook (deploy_website.yml):
   ---
- name: Install Nginx and deploy a basic website
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    document_root: /var/www/html

  tasks:
    - name: Ensure Nginx is installed
      package:
        name: nginx
        state: present

    - name: Create document root directory
      file:
        path: "{{ document_root }}"
        state: directory
        owner: root
        group: root
        mode: '0755'

    - name: Copy index.html file to the server
      copy:
        src: index.html
        dest: "{{ document_root }}/index.html"
        owner: root
        group: root
        mode: '0644'

    - name: Ensure Nginx is configured to listen on port {{ http_port }}
      lineinfile:
        path: /etc/nginx/sites-available/default
        regexp: '^listen '
        line: "listen {{ http_port }} default_server;"
      notify: restart nginx

    - name: Enable the default site
      file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link
      notify: restart nginx

    - name: Ensure Nginx service is running and enabled
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
