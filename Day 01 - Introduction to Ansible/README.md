# Day 1 – Introduction to Ansible

## 1. What is Ansible?

Ansible is an open-source automation tool used to automate IT tasks such as configuration management, application deployment and infrastructure provisioning.

In simple words, instead of connecting to every server manually and performing the same tasks, we can use Ansible to automate those tasks from a single machine.

**Example:**

Imagine an organization has 100 servers and we need to install Java on all of them.

- Without Ansible, we need to connect to every server and install Java manually.
- With Ansible, we can write a playbook and execute it from one machine. Ansible will connect to all the servers and install Java automatically.

This saves time and reduces manual work.

## 2. Why Do We Need Ansible?

To understand why Ansible was introduced, we first need to understand the work of System Administrators.

Earlier, organizations had System Administrators who were responsible for managing and maintaining servers.

Some of their responsibilities were:

- Keeping operating systems updated.
- Installing and updating software packages.
- Managing application dependencies like Java and Python.
- Configuring web servers like Apache and Nginx.
- Maintaining server security.
- Monitoring server resources like CPU and memory.

### What is Configuration Management?

Configuration Management means maintaining and managing the software, packages, dependencies and configurations of servers.

**Example:**

Imagine an organization has five servers:

| Server | Operating System |
|---|---|
| Server 1 | CentOS |
| Server 2 | Ubuntu |
| Server 3 | Debian |
| Server 4 | Alpine Linux |
| Server 5 | Windows |

The System Administrator needs to maintain all these servers.

The problem is that different operating systems use different commands.

For example, installing packages:

| Operating System | Package Manager |
|---|---|
| Ubuntu / Debian | APT |
| CentOS / RHEL | YUM / DNF |
| Alpine Linux | APK |
| Windows | Different installation methods |

If the administrator wants to install Java on all these servers, they need to understand the different commands and perform the installation separately.

Managing multiple servers manually becomes difficult and time-consuming.

## 3. How Did System Administrators Automate Tasks Earlier?

To reduce manual work, System Administrators started using scripting.

### A. Shell Scripting

Shell scripting is used to automate tasks in Linux.

Example:

```bash
#!/bin/bash

sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
```

This script updates packages, installs Apache and starts the Apache service on Ubuntu.

**Problem:** The same script may not work on CentOS or other Linux distributions because they use different package managers and commands.

### B. PowerShell

PowerShell is commonly used to automate administrative tasks in Windows.

So administrators had to use different scripts for different environments.

- Shell scripting for Linux.
- PowerShell for Windows.
- Different commands for different Linux distributions.

This made server management more complicated.

## 4. Configuration Management Tools

To solve these problems, configuration management tools were introduced.

Some popular tools are:

1. Puppet
2. Chef
3. SaltStack
4. Ansible

These tools help System Administrators automate server configurations.

### Challenges with Puppet and Chef

Puppet and Chef were popular configuration management tools, but they had some challenges.

**1. Learning Curve**

- Puppet has its own configuration language.
- Chef uses Ruby-based recipes and cookbooks.
- Administrators needed to learn their concepts and syntax.

**2. Agent-Based Architecture**

Traditionally, Puppet and Chef use agents on managed machines.

An agent is software installed on the target machine that communicates with the central management server.

For example, if an organization has 100 servers, administrators need to install and maintain agents on those servers.

This creates additional work.

## 5. Why Ansible Became Popular

Ansible made automation simpler by introducing two important features.

### A. YAML Syntax

Ansible uses YAML to write automation instructions called Playbooks.

YAML is simple and human-readable, so it is easier to understand and maintain.

Example:

```yaml
---
- name: Install Apache
  hosts: all
  become: yes

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
```

This playbook installs Apache on Ubuntu-based managed nodes.

### B. Agentless Architecture

Ansible follows an agentless architecture.

Agentless means we do not need to install Ansible software or an Ansible agent on every managed machine.

Ansible communicates with:

- Linux machines using SSH.
- Windows machines using WinRM or SSH.

For most Linux modules, Python is required on the managed nodes.

This makes Ansible easier to set up and manage.

## 6. How Ansible Works Internally

Ansible architecture mainly involves two important components:

### A. Control Node

A Control Node is the machine where Ansible is installed and from where automation tasks are executed.

For example, if I install Ansible on an Ubuntu EC2 instance, that EC2 instance becomes my Control Node.

### B. Managed Nodes

Managed Nodes are the remote machines that Ansible manages and automates.

For example, if I have 10 EC2 instances and want to install Java on all of them, those 10 instances are called Managed Nodes.

### Ansible Architecture

```text
             CONTROL NODE
           (Ansible Installed)
                   |
             Ansible Playbook
                   |
             SSH / WinRM
                   |
        -----------------------
        |          |          |
     Server 1   Server 2   Server 3
        |          |          |
      Linux      Linux     Windows
        |          |          |
     Java       Java      Software
    Installed  Installed  Installed
```

### How Does It Work?

1. Install Ansible on the Control Node.
2. Add the target servers to the Ansible Inventory.
3. Write an Ansible Playbook using YAML.
4. Execute the playbook from the Control Node.
5. Ansible connects to the Managed Nodes.
6. It performs the required tasks on those machines.

The main advantage is that we can manage multiple servers from one machine without installing an Ansible agent on every server.

## 7. What Can We Do Using Ansible?

Initially, Ansible was mainly used for Configuration Management. Today, it is a powerful automation platform used for different tasks.

### A. Configuration Management

Maintaining the desired configuration of servers.

Examples:

- Installing Java and Python.
- Installing Nginx and Apache.
- Updating software packages.
- Creating users.
- Managing services.

### B. Infrastructure Provisioning

Provisioning means creating infrastructure resources.

Ansible can communicate with cloud platforms and create resources.

Examples:

- Creating AWS EC2 instances.
- Creating S3 buckets.
- Creating Azure Virtual Machines.

Terraform is also widely used for infrastructure provisioning.

### C. Application Deployment

Ansible can automate application deployment.

For example, when deploying a Java application:

1. Connect to the target server.
2. Install required dependencies.
3. Copy the application JAR file.
4. Configure the application.
5. Start the application.

### D. Network Automation

Ansible can also automate network devices and configurations.

Examples:

- Configuring VLANs.
- Managing routers and switches.
- Automating network configurations.

## 8. Ansible vs Shell Scripting vs Python

It is important to understand when to use Shell scripting, Python and Ansible.

| Feature | Shell Scripting | Python | Ansible |
|---|---|---|---|
| Type | Scripting | Programming language | Automation tool |
| Syntax | Shell commands | Python code | YAML |
| Main purpose | Linux automation | General-purpose automation | IT infrastructure automation |
| Multiple server management | Requires additional scripting | Requires additional coding | Built-in support |
| Learning | Easy for basic tasks | Requires programming knowledge | Easy to get started |

### When to Use Shell Scripting?

Use Shell scripting for:

- Automating Linux commands.
- File operations.
- Managing Linux processes.
- Writing simple system scripts.

Example: Automatically deleting old log files.

### When to Use Python?

Use Python for:

- Working with APIs.
- Processing data.
- Building applications.
- Complex custom automation.

Example: Fetching AWS billing information using an API and generating a report.

### When to Use Ansible?

Use Ansible for:

- Managing multiple servers.
- Installing software across servers.
- Managing server configurations.
- Deploying applications.
- Automating infrastructure tasks.

Example: Installing Nginx on 50 EC2 instances.

**Easy way to remember:**

- Shell scripting – Automate Linux commands.
- Python – Build custom automation logic.
- Ansible – Automate infrastructure and server management.

## 9. What is an Ansible Playbook?

A Playbook is a YAML file that contains instructions for Ansible to execute.

It defines which servers to target and what tasks to perform on them.

Example:

```yaml
---
- name: Install Nginx
  hosts: all
  become: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
```

Explanation:

| Keyword | Meaning |
|---|---|
| name | Name of the play or task |
| hosts | Target machines |
| become | Execute tasks with elevated privileges |
| tasks | List of tasks to perform |
| apt | Ansible module for Debian-based package management |
| state: present | Ensures the package is installed |

## 10. Ansible Learning Roadmap – 14 Days

The instructor divided the complete Ansible series into 14 episodes.

| Day | Topics |
|---|---|
| Day 1 | Introduction to Ansible and Getting Started |
| Day 2 | Passwordless Authentication, Inventory and Ad Hoc Commands |
| Day 3 | First Ansible Playbook |
| Day 4 | Ansible Roles – Basics |
| Day 5 | Advanced Ansible Roles and Ansible Galaxy |
| Day 6 | Ansible Variables and Variable Precedence |
| Day 7 | Conditionals and Loops |
| Day 8 | Error Handling |
| Day 9 | Ansible Vault and Security |
| Day 10 | Policy as Code |
| Day 11 | Network Automation |
| Day 12 | Ansible Tower and Enterprise Automation |
| Day 13 | Terraform + Ansible Integration Project |
| Day 14 | Ansible Interview Questions |

### Important Concepts Coming in the Series

- **Passwordless Authentication:** Connecting to remote Linux servers using SSH keys without entering passwords every time.
- **Inventory:** A file containing the details of managed machines.
- **Ad Hoc Commands:** Executing simple one-time commands on managed nodes without writing a playbook.
- **Ansible Galaxy:** A platform for finding and reusing pre-built Ansible roles and collections.
- **Ansible Vault:** Encrypting sensitive information like passwords, API keys and secrets.
- **Ansible Tower:** An enterprise automation platform that provides a web interface, centralized management and role-based access control.

---

## 11. Interview Questions – Day 1

**1. What is Ansible?**

Ansible is an open-source automation tool used for configuration management, application deployment, infrastructure provisioning and other IT automation tasks.

**2. Why is Ansible called agentless?**

Ansible is called agentless because we do not need to install an Ansible agent on managed nodes. It communicates with Linux machines using SSH and Windows machines using WinRM or SSH.

**3. What is a Control Node?**

A Control Node is the machine where Ansible is installed and from where automation tasks are executed.

**4. What are Managed Nodes?**

Managed Nodes are the remote machines that Ansible manages and automates.

**5. Which language does Ansible use?**

Ansible Playbooks are written in YAML. Ansible itself is primarily implemented in Python.

**6. Does Ansible support Windows?**

Yes. Ansible supports Windows managed nodes using WinRM or SSH.

**7. What is the difference between Ansible and Terraform?**

Terraform is primarily used for infrastructure provisioning, whereas Ansible is primarily used for configuration management and application deployment. Both tools can be used together.

---

## 12. Day 1 – Quick Revision

| Concept | Meaning |
|---|---|
| Ansible | Open-source automation tool |
| Configuration Management | Maintaining server configurations |
| YAML | Syntax used to write Ansible Playbooks |
| Control Node | Machine where Ansible is installed |
| Managed Nodes | Machines managed by Ansible |
| Agentless | No Ansible agent required on managed nodes |
| SSH | Common connection method for Linux |
| WinRM | Common connection method for Windows |
| Inventory | List of managed nodes |
| Playbook | YAML file containing automation instructions |
| Ansible Galaxy | Reusable roles and collections |
| Ansible Vault | Encrypts sensitive information |

## What I Learned Today

Today I started learning Ansible and understood why automation is important in DevOps.

I learned how System Administrators managed servers earlier, the problems with manual configuration management and how tools like Puppet, Chef and Ansible helped solve those problems.

I also understood the Ansible architecture, Control Nodes, Managed Nodes and its agentless approach.

Finally, I learned the difference between Shell scripting, Python and Ansible and understood how Ansible is used for configuration management, infrastructure provisioning, application deployment and network automation.

**Key Takeaway:** Ansible helps us automate and manage multiple servers from a single Control Node without manually connecting to every server or installing an Ansible agent on each managed node.
