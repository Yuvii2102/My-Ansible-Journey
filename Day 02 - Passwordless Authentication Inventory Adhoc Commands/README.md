# Day 2 – Passwordless Authentication, Inventory and Ad Hoc Commands

Today I learned three important concepts in Ansible:

1. Passwordless Authentication
2. Ansible Inventory
3. Ansible Ad Hoc Commands

These concepts help Ansible connect to remote servers and execute automation tasks.

---

## 1. Passwordless Authentication

### What is Passwordless Authentication?

Passwordless Authentication is a method that allows one machine to connect to another machine without entering a password every time.

In Ansible, we have two important components:

- **Control Node:** The machine where Ansible is installed.
- **Managed Nodes:** The remote machines that Ansible manages.

When Ansible connects to Managed Nodes, it needs authentication.

There are two common authentication methods:

1. Password Authentication
2. SSH Key Authentication

If Ansible asks for authentication every time it connects to a server, automation becomes difficult.

To solve this problem, we configure Passwordless Authentication.

### Why Do We Need Passwordless Authentication?

Imagine we have one Control Node and four Managed Nodes.

We want to install Nginx on all four servers using Ansible.

Without passwordless authentication, we may need to provide authentication details whenever Ansible connects to the servers.

This becomes difficult when:

- Managing multiple servers.
- Running automation through Jenkins.
- Scheduling automation using Cron jobs.
- Executing playbooks automatically.

With passwordless authentication, the Control Node can connect to Managed Nodes without entering the password every time.

### How Does It Work?

Imagine Machine A wants to connect to Machine B.

Initially, we configure authentication between these machines.

After successful authentication setup, Machine B trusts the configured public key from Machine A.

Now Machine A can connect to Machine B without entering the password every time.

This is called Passwordless Authentication.

### Two Ways to Configure Passwordless Authentication

1. Using SSH Keys.
2. Using Password Authentication.

---

## 2. Passwordless Authentication Using SSH Keys

SSH stands for Secure Shell. It allows us to securely connect to remote machines.

SSH uses two keys:

- **Public Key:** Stored on the Managed Node.
- **Private Key:** Kept securely on the Control Node.

The public and private keys work together to authenticate the connection.

### How Does It Work?

1. The Control Node generates an SSH key pair.
2. We copy the public key to the Managed Node.
3. The Managed Node stores the public key in the `authorized_keys` file.
4. When the Control Node connects, SSH checks the key.
5. If authentication is successful, the connection is established without asking for a password.

### Practical Steps

**Step 1: Generate SSH Keys**

On the Control Node, execute:

```bash
ssh-keygen -t ed25519
```

This generates two files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

- `id_ed25519` – Private key.
- `id_ed25519.pub` – Public key.

Never share your private key.

**Step 2: Copy the Public Key to the Managed Node**

Execute:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@MANAGED_NODE_IP
```

Replace `MANAGED_NODE_IP` with the IP address of your target machine.

If you are using an AWS EC2 instance and already have a PEM file, use:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub -o IdentityFile=/path/to/aws-key.pem ubuntu@MANAGED_NODE_IP
```

The existing PEM file is used for initial authentication.

**Step 3: Test the Connection**

```bash
ssh ubuntu@MANAGED_NODE_IP
```

If everything is configured correctly, you should connect without specifying the PEM file or entering a password.

### Important AWS Note

AWS EC2 instances normally use SSH key authentication by default.

We can use the existing PEM file to connect initially and configure our own SSH public key for future passwordless connections.

---

## 3. Passwordless Authentication Using Password

Another method is to configure password-based authentication.

This method is useful when an organization allows remote login using passwords.

### Practical Steps

**Step 1: Connect to the Managed Node**

```bash
ssh -i aws-key.pem ubuntu@MANAGED_NODE_IP
```

**Step 2: Enable Password Authentication**

Open the SSH server configuration:

```bash
sudo vim /etc/ssh/sshd_config
```

Find:

```text
#PasswordAuthentication no
```

Update it to:

```text
PasswordAuthentication yes
```

Make sure no other SSH configuration overrides this setting.

**Step 3: Set a Password for the User**

```bash
sudo passwd ubuntu
```

Enter and confirm the password.

**Step 4: Restart the SSH Service**

On Ubuntu:

```bash
sudo systemctl restart ssh
```

**Step 5: Copy the Public Key**

From the Control Node, execute:

```bash
ssh-copy-id ubuntu@MANAGED_NODE_IP
```

It will ask for the user's password.

Enter the password you created.

The public key will be copied to the Managed Node.

**Step 6: Test the Connection**

```bash
ssh ubuntu@MANAGED_NODE_IP
```

Now you should be able to connect without entering the password.

**Security Note:** Password authentication is more exposed to brute-force attacks. SSH key-based authentication is generally preferred for production servers.

### Difference Between the Two Methods

| SSH Key Authentication | Password Authentication |
|---|---|
| Uses public and private keys | Uses a user password initially |
| Public key is copied to the server | Public key is copied after password authentication |
| Generally more secure | More exposed to password attacks |
| Commonly used with AWS EC2 | Depends on organizational configuration |

### What Did We Achieve?

Our Control Node can now connect to Managed Nodes without entering a password every time.

This is an important prerequisite for running Ansible automation.

---

## 4. What is Ansible Inventory?

### Definition

Ansible Inventory is a file that contains information about the Managed Nodes that Ansible needs to manage.

In simple words, Inventory tells Ansible:

- Which servers should be managed?
- What are their IP addresses?
- Which username should Ansible use?
- Which group does each server belong to?

Without an inventory, Ansible will not know which machines it needs to manage.

### Example

Imagine an organization has four servers:

| Server | IP Address | Purpose |
|---|---|---|
| Server 1 | 192.168.1.10 | Application |
| Server 2 | 192.168.1.11 | Application |
| Server 3 | 192.168.1.12 | Database |
| Server 4 | 192.168.1.13 | Database |

We can add these servers to an Ansible Inventory file.

### Creating an Inventory File Using YAML

Ansible supports YAML format for inventory files.

First, create a file:

```bash
vim inventory.yaml
```

Add the following configuration:

```yaml
all:
  hosts:
    server1:
      ansible_host: 192.168.1.10
      ansible_user: ubuntu

    server2:
      ansible_host: 192.168.1.11
      ansible_user: ubuntu

    server3:
      ansible_host: 192.168.1.12
      ansible_user: ubuntu

    server4:
      ansible_host: 192.168.1.13
      ansible_user: ubuntu
```

### Understanding the YAML Inventory

- `all` – Contains all the servers.
- `hosts` – Contains the list of individual servers.
- `server1` – Name given to the server.
- `ansible_host` – IP address of the server.
- `ansible_user` – Username used to connect to the server.

**Important:** `server1` is just a name we give to identify the machine. `ansible_host` contains its actual IP address.

### Grouping Servers in Inventory

In real-world organizations, we may have different types of servers.

For example:

- Application Servers
- Database Servers

Instead of managing all servers together, we can create groups.

Create or update `inventory.yaml`:

```yaml
all:
  children:
    app:
      hosts:
        app1:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu

        app2:
          ansible_host: 192.168.1.11
          ansible_user: ubuntu

    db:
      hosts:
        db1:
          ansible_host: 192.168.1.12
          ansible_user: ubuntu

        db2:
          ansible_host: 192.168.1.13
          ansible_user: ubuntu
```

### Understanding the Groups

- `all` – Parent group containing all servers.
- `children` – Contains other groups.
- `app` – Application Server group.
- `db` – Database Server group.
- `hosts` – Contains the servers inside each group.

Now we can execute commands on specific groups.

**Execute on all servers:**

```bash
ansible -i inventory.yaml all -m ping
```

**Execute only on Application Servers:**

```bash
ansible -i inventory.yaml app -m ping
```

**Execute only on Database Servers:**

```bash
ansible -i inventory.yaml db -m ping
```

### Default Inventory Location

Ansible has a default inventory location:

```text
/etc/ansible/hosts
```

We can add our Managed Nodes to this file.

If the directory or file does not exist, we can create it.

When we use the default inventory, we don't need to specify `-i` every time.

Example:

```bash
ansible all -m ping
```

### Custom Inventory vs Default Inventory

**Custom Inventory:**

We create an inventory file for a particular project.

Example:

```text
ansible-project/
├── inventory.yaml
└── playbook.yaml
```

Execute it using:

```bash
ansible -i inventory.yaml all -m ping
```

**Default Inventory:**

We add servers to:

```text
/etc/ansible/hosts
```

Execute it using:

```bash
ansible all -m ping
```

### Recommended Approach

For organizations with multiple projects, it is better to maintain separate inventory files for different projects.

This makes server management easier and gives each project control over its own Managed Nodes.

---

## 5. What are Ansible Ad Hoc Commands?

Ansible provides two main ways to execute automation tasks:

1. Ansible Playbooks.
2. Ansible Ad Hoc Commands.

### Definition

An Ad Hoc Command is a one-time command used to perform a simple task on one or more Managed Nodes without creating a Playbook.

In simple words, if we want to perform a small task quickly, we can use an Ad Hoc Command instead of writing a complete YAML file.

### Example

Imagine we want to:

- Check connectivity between servers.
- Create a file.
- Install a package.
- Restart a service.
- Check disk space.

We can perform these tasks using Ad Hoc Commands.

### Syntax of Ansible Ad Hoc Commands

```bash
ansible <host-pattern> -i <inventory-file> -m <module> -a "<arguments>"
```

Let's understand each part:

| Component | Meaning |
|---|---|
| `ansible` | Ansible command |
| `-i` | Specifies the inventory file |
| `host-pattern` | Specifies which servers to target |
| `-m` | Specifies the module |
| `-a` | Specifies arguments for the module |

### Example 1: Check Server Connectivity

```bash
ansible -i inventory.yaml all -m ping
```

Explanation:

- `-i inventory.yaml` – Uses our inventory file.
- `all` – Executes on all servers.
- `-m ping` – Uses the Ansible ping module.

Expected output:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

If we receive `pong`, it means Ansible successfully communicated with that server.

**Note:** Ansible's ping module checks whether Ansible can connect and execute a basic module. It is different from the normal Linux network ping command.

### Example 2: Execute a Command on One Server

```bash
ansible -i inventory.yaml app1 -m ping
```

This executes the ping module only on the server named `app1`.

### Example 3: Execute a Command on a Group

```bash
ansible -i inventory.yaml app -m ping
```

This executes the ping module only on Application Servers.

### Example 4: Execute a Shell Command

We can use the `shell` module to execute Linux shell commands.

```bash
ansible -i inventory.yaml all -m shell -a "ls /etc"
```

This lists the files and directories inside `/etc` on all Managed Nodes.

### Example 5: Check Disk Space

```bash
ansible -i inventory.yaml all -m shell -a "df -h"
```

This displays disk usage on all Managed Nodes.

### Example 6: Check Memory Usage

```bash
ansible -i inventory.yaml all -m shell -a "free -h"
```

This displays memory usage on all Managed Nodes.

### Example 7: Install a Package

We can use the `apt` module to install packages on Ubuntu and Debian servers.

```bash
ansible -i inventory.yaml all -m apt -a "name=nginx state=present" -b
```

Explanation:

- `apt` – Package management module.
- `name=nginx` – Package to install.
- `state=present` – Ensures Nginx is installed.
- `-b` – Runs the task with elevated privileges using sudo.

### Example 8: Create a File

```bash
ansible -i inventory.yaml all -m file -a "path=/tmp/test.txt state=touch"
```

This creates a file named `test.txt` on all Managed Nodes.

### Example 9: Copy a File

```bash
ansible -i inventory.yaml all -m copy -a "src=test.txt dest=/tmp/test.txt"
```

This copies the local file `test.txt` from the Control Node to all Managed Nodes.

### Example 10: Check Running Services

```bash
ansible -i inventory.yaml all -m shell -a "systemctl status nginx"
```

This checks the status of Nginx on all Managed Nodes.

---

## 6. What is an Ansible Module?

A Module is a small piece of code that Ansible uses to perform a particular task on Managed Nodes.

Some commonly used modules are:

| Module | Purpose |
|---|---|
| `ping` | Check Ansible connectivity |
| `shell` | Execute shell commands |
| `command` | Execute commands on remote machines |
| `apt` | Manage packages on Debian-based systems |
| `yum` | Manage packages on older RHEL-based systems |
| `dnf` | Manage packages on modern RHEL-based systems |
| `file` | Create, delete and manage files |
| `copy` | Copy files to remote machines |
| `service` | Manage services |
| `user` | Create and manage users |

We can explore more modules in the official Ansible documentation.

---

## 7. When Should We Use Ad Hoc Commands?

Ad Hoc Commands are useful when we need to perform simple, one-time tasks.

Some common use cases:

**1. Connectivity Checking**

Checking whether Ansible can communicate with Managed Nodes.

**2. Package Management**

Installing or updating packages across multiple servers.

**3. File Management**

Creating, deleting and changing file permissions.

**4. Service Management**

Starting, stopping and restarting services.

**5. Server Information**

Checking disk space, memory usage and system information.

---

## 8. Ansible Ad Hoc Commands vs Playbooks

Both are used to execute tasks, but they serve different purposes.

| Ad Hoc Commands | Playbooks |
|---|---|
| Used for simple tasks | Used for multiple tasks |
| Executed directly from the command line | Written in YAML files |
| Mostly used for one-time operations | Can be reused multiple times |
| Not stored as a playbook file | Can be stored in Git |
| Useful for quick operations | Useful for organized automation |

### What is a Basic Ansible Playbook?

A Playbook is a YAML file containing instructions for Ansible to execute.

For example, if we want to install Nginx on all Application Servers, we can write a simple playbook.

**Step 1: Create a Playbook**

```bash
vim install-nginx.yaml
```

**Step 2: Add the Following Content**

```yaml
---
- name: Install Nginx
  hosts: app
  become: yes

  tasks:
    - name: Install Nginx package
      apt:
        name: nginx
        state: present
```

**Step 3: Execute the Playbook**

```bash
ansible-playbook -i inventory.yaml install-nginx.yaml
```

### Understanding the Playbook

| Keyword | Meaning |
|---|---|
| `---` | Marks the beginning of a YAML document |
| `name` | Name of the play or task |
| `hosts` | Servers where the playbook should run |
| `become` | Runs tasks with elevated privileges |
| `tasks` | List of tasks to execute |
| `apt` | Package management module |
| `name: nginx` | Package we want to install |
| `state: present` | Ensures Nginx is installed |

### Difference Between Ad Hoc Commands and Playbooks

**Ad Hoc Command:**

```bash
ansible -i inventory.yaml app -m ping
```

Used for quickly checking connectivity.

**Playbook:**

```yaml
---
- name: Install Nginx
  hosts: app
  become: yes

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
```

Used when we want to organize and reuse automation tasks.

**My Understanding:**

- Ad Hoc Commands are useful for quick, one-time tasks.
- Playbooks are useful for multiple tasks and reusable automation.

As a DevOps Engineer, we should understand both.

---

## 9. Day 2 – Quick Revision

| Concept | Meaning |
|---|---|
| Passwordless Authentication | Connect to servers without entering a password every time |
| SSH | Secure Shell protocol |
| Public Key | Key copied to the Managed Node |
| Private Key | Key kept securely on the Control Node |
| `ssh-copy-id` | Copies the public key to a remote server |
| Inventory | Contains details of Managed Nodes |
| YAML Inventory | Inventory written using YAML syntax |
| `/etc/ansible/hosts` | Common default inventory location |
| `app` | Application Server group |
| `db` | Database Server group |
| Ad Hoc Command | One-time command executed through Ansible |
| `-i` | Specifies inventory |
| `-m` | Specifies module |
| `-a` | Specifies module arguments |
| `all` | Targets all hosts in inventory |
| `-b` | Enables privilege escalation |
| Playbook | YAML file containing automation instructions |

---

## What I Learned Today

Today I learned how Ansible connects to remote servers using Passwordless Authentication.

I understood the difference between SSH key-based authentication and password-based authentication and why passwordless authentication is important for automation.

I also learned about Ansible Inventory using YAML format. I understood how to add servers to an inventory file and group them into Application and Database Servers.

Finally, I learned Ansible Ad Hoc Commands and how to use modules like ping, shell, apt, file and copy to perform simple tasks on multiple servers.

I also got introduced to basic Ansible Playbooks and learned how to install Nginx using a simple YAML file.

**Key Takeaway:** Passwordless Authentication allows Ansible to connect to remote servers without entering passwords every time. Inventory tells Ansible which servers to manage, Ad Hoc Commands help execute quick tasks and Playbooks help organize reusable automation.
