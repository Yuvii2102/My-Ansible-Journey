# SSH Authentication in Ansible – Practical Summary

## 1. Introduction

Before working with Ansible, we need to establish communication between the Control Node and Managed Nodes.

Ansible uses SSH (Secure Shell) to connect to remote Linux servers and execute tasks.

In this lab, we configured two types of SSH authentication:

1. SSH Key-Based Passwordless Authentication
2. Password-Based SSH Authentication

## 2. Our Lab Setup

We created three Ubuntu EC2 instances.

| Instance | Role | Private IP |
|---|---|---|
| EC2 Instance 1 | Control Node | 172.31.28.28 |
| EC2 Instance 2 | Managed Node 1 | 172.31.31.231 |
| EC2 Instance 3 | Managed Node 2 | 172.31.26.245 |

**Username:** ubuntu

---

## 3. Practical 1: SSH Key-Based Passwordless Authentication

<badge color="success">Completed</badge>

### What did we do?

1. Generated an SSH key pair on the Control Node.
2. Created a private key and public key.
3. Copied the public key to both Managed Nodes using `ssh-copy-id`.
4. Configured the required SSH permissions.
5. Tested SSH connections without entering passwords.

### What did we achieve?

Our Control Node can connect to both Managed Nodes without repeatedly entering an SSH login password or providing the original AWS PEM key.

### Architecture

```text
           CONTROL NODE
            172.31.28.28
                  |
                  |
          SSH Key Authentication
                  |
             /         \
            /           \
           v             v
     MANAGED NODE 1   MANAGED NODE 2
      172.31.31.231    172.31.26.245
```

**Result:** Both Managed Nodes successfully accepted SSH connections using the generated private key.

---

## 4. Practical 2: Password-Based SSH Authentication

<badge color="success">Completed</badge>

### What did we do?

1. Set passwords for the Ubuntu users on both Managed Nodes.
2. Enabled `PasswordAuthentication yes`.
3. Validated SSH configuration using `sshd -t`.
4. Restarted the SSH service.
5. Tested connections using password authentication while disabling public key authentication for the test.

### What did we achieve?

Our Control Node can also connect to both Managed Nodes using their usernames and passwords.

### Architecture

```text
           CONTROL NODE
            172.31.28.28
                  |
                  |
          Password Authentication
                  |
             /         \
            /           \
           v             v
     MANAGED NODE 1   MANAGED NODE 2
      172.31.31.231    172.31.26.245
```

**Result:** Both Managed Nodes successfully accepted SSH connections using their Ubuntu usernames and passwords.

---

## 5. Difference Between Both Practical Sessions

| Feature | Practical 1 | Practical 2 |
|---|---|---|
| Authentication | SSH keys | Password |
| Private key | Required on client | Not required |
| Password entry | Not required for login | Required |
| Security | Generally stronger | Depends on password strength and configuration |
| Ansible usage | Commonly preferred | Supported |

### Simple explanation

- **Key-based authentication:** The Control Node uses its private key to authenticate with the public key stored on the Managed Node.
- **Password-based authentication:** The Control Node connects using a username and password.

Both methods allow the Control Node to communicate with remote Linux servers through SSH.

---

## 6. Most Important: Why Are We Doing This for Ansible?

Imagine you have:

- 1 Control Node
- 2 Managed Nodes
- 100 Managed Nodes in a real company

You want to install Docker on all 100 servers.

### Without Ansible

You might have to connect to every server manually and execute commands.

For example:

1. Connect to Server 1.
2. Install Docker.
3. Connect to Server 2.
4. Install Docker.
5. Repeat the same process for all 100 servers.

This takes more time and effort.

### With Ansible

1. The Control Node connects to Managed Nodes using SSH.
2. Ansible executes tasks remotely.
3. You manage multiple servers from one place.

### Example

Suppose we want to install Docker on both Managed Nodes.

Instead of manually connecting to each server, we can write an Ansible playbook to install Docker on both machines.

Ansible connects through SSH and executes the required tasks.

**That's why SSH authentication is an important prerequisite for Ansible.**

---

## 7. What Happens Behind the Scenes?

When we execute an Ansible command:

```bash
ansible all -i inventory -m ping
```

Ansible:

1. Reads the Managed Nodes from the inventory file.
2. Uses SSH to connect to the remote servers.
3. Authenticates using the configured SSH authentication method.
4. Executes the requested module.
5. Returns the result to the Control Node.

**Note:** Ansible's `ping` module is used to check connectivity and Python availability. It is not the same as the network ICMP ping command.

---

## 8. Final Outcome

We successfully completed both SSH authentication practicals.

### Practical 1: Key-Based Authentication

- Generated SSH key pairs.
- Copied public keys to both Managed Nodes.
- Configured SSH permissions.
- Successfully tested passwordless SSH connections.

### Practical 2: Password-Based Authentication

- Set Ubuntu user passwords.
- Enabled password authentication.
- Validated SSH configuration.
- Restarted the SSH service.
- Successfully tested password-based SSH connections.

### Overall Achievement

We established two ways for the Ansible Control Node to authenticate and communicate with both Managed Nodes.

This provides the foundation for automating remote server management using Ansible.

---

## 9. Interview-Ready Explanation

We configured two types of SSH authentication between an Ansible Control Node and two Managed Nodes.

First, we implemented SSH key-based authentication by generating a public-private key pair on the Control Node and copying the public key to both Managed Nodes.

Next, we enabled password-based authentication by configuring the SSH server and setting passwords for the Ubuntu users.

We successfully tested both authentication methods. This helped us understand how Ansible connects to remote Linux servers through SSH and executes tasks remotely.

---

**Conclusion:** We successfully established SSH communication between the Control Node and Managed Nodes using key-based and password-based authentication. This is the foundation for automating server management with Ansible.
