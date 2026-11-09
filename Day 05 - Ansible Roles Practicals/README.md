# Part 2: Practical Implementation

## 1. Project Overview

In this practical, I deployed a static website using Apache by creating and using an Ansible Role.

Instead of writing all the automation tasks directly inside a playbook, I organized them into a separate role named `httpd`.

This practical helped me understand how Ansible Roles make automation more organized, reusable, and maintainable.

### Objective

- Create an Ansible Role using Ansible Galaxy.
- Organize Apache deployment tasks inside the role.
- Move static website files into the role.
- Create a playbook to execute the role.
- Configure the inventory for the managed node.
- Deploy and verify the Apache web server.

## 2. Infrastructure Setup

I used two AWS EC2 Ubuntu instances.

| Component | Description |
|---|---|
| Control Node | EC2 instance where Ansible is installed |
| Managed Node | EC2 instance where Apache is deployed |
| Automation Tool | Ansible |
| Web Server | Apache2 |
| Operating System | Ubuntu |
| Authentication | SSH Key-Based Authentication |

### Infrastructure Details

- Control Node: `172.31.24.9`
- Managed Node: `172.31.26.36`
- Managed Node SSH User: `ubuntu`

**Note:** These are the private IP addresses used in my practical. If I recreate the instances, I need to update the inventory with the new managed node private IP.

## 3. Navigate to the Project Directory

First, I connected to my Ansible Control Node and navigated to my existing project directory.

```bash
cd ~/My-Complete-DevOps-Projects
```

Then I entered the Project 15 directory:

```bash
cd "15 - Ansible Roles Practicals"
```

I checked the existing files:

```bash
ls -la
```

## 4. Create an Ansible Role

Ansible provides the `ansible-galaxy` command to create roles with a predefined directory structure.

I created a role named `httpd`:

```bash
ansible-galaxy role init httpd
```

This command automatically generated the standard directory structure.

I verified the structure using:

```bash
ls -R httpd
```

The generated structure looked like this:

```text
httpd/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
├── vars/
│   └── main.yml
└── README.md
```

## 5. Organize the Website File

I already had an `index.html` file from my previous Apache deployment practical.

Instead of keeping it in the project root, I moved it into the `files/` directory of the `httpd` role.

```bash
mv index.html httpd/files/
```

I verified the file location:

```bash
ls httpd/files/
```

Expected output:

```text
index.html
```

The `files/` directory stores static files that the role can copy to the managed node.

## 6. Configure the Ansible Role Tasks

Next, I opened the main task file inside the role.

```bash
vim httpd/tasks/main.yml
```

I added the following tasks:

```yaml
---
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: true

- name: Copy website file
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html

- name: Start and enable Apache
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: true
```

### Understanding the Tasks

**Task 1: Install Apache**

```yaml
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: true
```

- `ansible.builtin.apt` – Uses the APT package manager.
- `name: apache2` – Specifies the Apache package.
- `state: present` – Ensures Apache is installed.
- `update_cache: true` – Updates the package cache before installation.

**Task 2: Copy Website File**

```yaml
- name: Copy website file
  ansible.builtin.copy:
    src: index.html
    dest: /var/www/html/index.html
```

- `ansible.builtin.copy` – Copies a file from the control node to the managed node.
- `src: index.html` – Refers to the file inside the role's `files/` directory.
- `dest` – Specifies the destination path on the managed node.

**Task 3: Start and Enable Apache**

```yaml
- name: Start and enable Apache
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: true
```

- `ansible.builtin.service` – Manages system services.
- `name: apache2` – Specifies the Apache service.
- `state: started` – Ensures the service is running.
- `enabled: true` – Ensures Apache starts automatically when the system boots.

## 7. Create the Role-Based Playbook

After configuring the role, I created the main playbook that calls the `httpd` role.

I opened the playbook:

```bash
vim first-playbook.yaml
```

I added:

```yaml
---
- name: Deploy Static Website Using Ansible Roles
  hosts: all
  become: true

  roles:
    - httpd
```

### Understanding the Playbook

- `name` – Describes the purpose of the playbook.
- `hosts: all` – Executes the play against all hosts in the selected inventory.
- `become: true` – Uses privilege escalation to execute administrative tasks.
- `roles` – Specifies the roles to execute.
- `httpd` – Calls the role created earlier.

Ansible automatically loads the tasks from:

```text
httpd/tasks/main.yml
```

This means I do not need to write all the Apache installation and deployment tasks again inside the playbook.

## 8. Configure the Inventory File

Next, I configured the inventory file to identify the managed node.

I copied the example inventory file:

```bash
cp inventory.example.ini inventory.ini
```

I opened it:

```bash
vim inventory.ini
```

I added the managed node's private IP address:

```ini
[webservers]
172.31.26.36 ansible_user=ubuntu
```

### Understanding the Inventory

- `[webservers]` – Defines a group of managed hosts.
- `172.31.26.36` – Private IP address of the managed node.
- `ansible_user=ubuntu` – Specifies the SSH user used to connect to the managed node.

**Important:** The Control Node must have SSH connectivity to the Managed Node, and the appropriate SSH key authentication must already be configured.

## 9. Protect the Inventory File

The inventory file contains environment-specific information such as server IP addresses.

I added the following entry to the `.gitignore` file:

```text
inventory.ini
```

This prevents the local inventory file from being accidentally tracked and pushed to GitHub.

I kept `inventory.example.ini` in the repository as a reference for creating the inventory file again.

## 10. Verify Ansible Connectivity

Before running the playbook, I checked whether the Control Node could communicate with the Managed Node.

```bash
ansible all -i inventory.ini -m ping
```

Expected successful output:

```text
172.31.26.36 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

The `ping` module verifies Ansible connectivity to the managed node. It is not an ICMP network ping.

## 11. Execute the Ansible Playbook

Once the inventory and role were ready, I executed the playbook.

```bash
ansible-playbook -i inventory.ini first-playbook.yaml
```

During execution, Ansible performed the following operations:

1. Connected to the managed node.
2. Installed Apache2.
3. Copied `index.html` to `/var/www/html/index.html`.
4. Started the Apache service.
5. Enabled Apache to start automatically during system boot.

Ansible displayed the execution results for each task.

## 12. Verify the Deployment

After executing the playbook, I connected to the managed node and verified the Apache service.

### Check Apache Service Status

```bash
sudo systemctl status apache2
```

The expected status is:

```text
Active: active (running)
```

### Verify the Website Locally

```bash
curl -I http://localhost
```

Expected response:

```text
HTTP/1.1 200 OK
```

### Access the Website Through a Browser

I opened the managed node's public IPv4 address in my browser:

```text
http://MANAGED_NODE_PUBLIC_IP
```

The website displayed the content from my `index.html` file.

**Note:** The managed node's security group must allow inbound HTTP traffic on TCP port 80 for browser access.

## 13. Final Project Structure

After completing the practical, my project directory looked like this:

```text
15 - Ansible Roles Practicals/
│
├── httpd/
│   ├── defaults/
│   │   └── main.yml
│   │
│   ├── files/
│   │   └── index.html
│   │
│   ├── handlers/
│   │   └── main.yml
│   │
│   ├── meta/
│   │   └── main.yml
│   │
│   ├── tasks/
│   │   └── main.yml
│   │
│   ├── templates/
│   │
│   ├── tests/
│   │   ├── inventory
│   │   └── test.yml
│   │
│   ├── vars/
│   │   └── main.yml
│   │
│   └── README.md
│
├── first-playbook.yaml
├── inventory.example.ini
├── inventory.ini
├── .gitignore
└── README.md
```

The `inventory.ini` file is kept locally and excluded from Git tracking.

## 14. Final Outcome

Successfully created an Ansible Role named `httpd` and used it to automate the deployment of a static website using Apache.

Through this practical, I learned how to:

- Create roles using Ansible Galaxy.
- Organize automation tasks into reusable components.
- Use the `files/` directory to manage static files.
- Configure tasks using Ansible modules.
- Execute roles through a playbook.
- Configure inventories for managed nodes.
- Deploy and verify an Apache web server using Ansible.

**Key Learning:** Ansible Roles help convert large playbooks into smaller, reusable, and well-organized automation components.

---

**End of Project 15 – Ansible Roles Practicals**
