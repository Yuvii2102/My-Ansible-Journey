# Practical 3: Ansible Inventory Configuration

## 1. Objective

Create and configure an Ansible inventory file to manage multiple remote Linux servers from a single Control Node.

The inventory file tells Ansible:

- Which servers it needs to manage.
- The IP addresses of the Managed Nodes.
- Which username to use for SSH connections.
- Which private key to use for authentication.
- How to organize servers into groups.

## 2. Lab Architecture

We are using three Ubuntu EC2 instances.

| Instance | Role | Private IP |
|---|---|---|
| EC2 Instance 1 | Control Node | 172.31.28.28 |
| EC2 Instance 2 | Managed Node 1 | 172.31.31.231 |
| EC2 Instance 3 | Managed Node 2 | 172.31.26.245 |

**Username:** ubuntu

### Architecture

```text
              CONTROL NODE
               172.31.28.28
                    |
                    |
             Ansible Inventory
                inventory.ini
                    |
              SSH Connection
                    |
               /         \
              /           \
             v             v
      MANAGED NODE 1   MANAGED NODE 2
       172.31.31.231    172.31.26.245
```

## 3. Prerequisites

Before starting, ensure:

- Ansible is installed on the Control Node.
- Both Managed Nodes are running.
- SSH key-based authentication is configured.
- Both Managed Nodes are accessible from the Control Node.
- Python 3 is installed on the Managed Nodes.

## 4. Step 1: Create an Ansible Project Directory

First, connect to your Control Node.

Create a directory for Ansible practicals:

```bash
mkdir ansible-practicals
```

Navigate into the directory:

```bash
cd ansible-practicals
```

Check your current location:

```bash
pwd
```

Expected output:

```text
/home/ubuntu/ansible-practicals
```

**Why do we create this directory?**

It helps us organize our Ansible practicals, inventory files and playbooks in one place.

## 5. Step 2: Create an Inventory File

An inventory file contains the details of the servers that Ansible manages.

Create a file named `inventory.ini`:

```bash
touch inventory.ini
```

Open the file:

```bash
vim inventory.ini
```

Add the following configuration:

```ini
[managed]
node1 ansible_host=172.31.31.231
node2 ansible_host=172.31.26.245

[managed:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

Save and exit:

```text
:wq
```

## 6. Understanding the Inventory File

Let's understand each configuration.

### [managed]

```ini
[managed]
```

This creates a group named `managed`.

We can use this group name to execute Ansible commands on all servers inside it.

### node1 and node2

```ini
node1 ansible_host=172.31.31.231
node2 ansible_host=172.31.26.245
```

- `node1` is the name assigned to Managed Node 1.
- `node2` is the name assigned to Managed Node 2.
- `ansible_host` specifies the actual IP address of each server.

### [managed:vars]

```ini
[managed:vars]
```

This defines variables that apply to all servers inside the `managed` group.

### ansible_user

```ini
ansible_user=ubuntu
```

Specifies the username Ansible uses to connect to the Managed Nodes.

### ansible_ssh_private_key_file

```ini
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

Specifies the private SSH key used for authentication.

**Important:** The private key stays on the Control Node.

## 7. Step 3: Verify the Inventory File

Display the inventory configuration:

```bash
cat inventory.ini
```

Expected output:

```ini
[managed]
node1 ansible_host=172.31.31.231
node2 ansible_host=172.31.26.245

[managed:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

## 8. Step 4: Check Inventory Using Ansible

Ansible provides a command called `ansible-inventory` to inspect inventory information.

### Display inventory in JSON format

```bash
ansible-inventory -i inventory.ini --list
```

This displays the inventory details in JSON format.

### Display inventory as a tree

```bash
ansible-inventory -i inventory.ini --graph
```

Expected output:

```text
@all:
  |--@managed:
  |  |--node1
  |  |--node2
```

**What did we verify?**

Ansible successfully recognized both Managed Nodes under the `managed` group.

## 9. Step 5: Test Connectivity Using Ansible Ping

Now we will check whether the Control Node can communicate with both Managed Nodes.

Execute:

```bash
ansible managed -i inventory.ini -m ping
```

### Understanding the command

| Option | Meaning |
|---|---|
| `ansible` | Runs an Ansible ad-hoc command |
| `managed` | Targets the managed group |
| `-i inventory.ini` | Specifies the inventory file |
| `-m ping` | Executes the Ansible ping module |

### Expected output

```text
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### What does pong mean?

When Ansible returns `pong`, it means the Ansible ping module successfully communicated with the remote server.

**Important:** Ansible's ping module is different from the normal Linux network ping command. It checks Ansible connectivity and remote Python availability.

## 10. Step 6: Test Individual Managed Nodes

We can also test each Managed Node separately.

### Test Managed Node 1

```bash
ansible node1 -i inventory.ini -m ping
```

### Test Managed Node 2

```bash
ansible node2 -i inventory.ini -m ping
```

Both nodes should return:

```text
"ping": "pong"
```

This confirms that each Managed Node is reachable individually through Ansible.

## 11. Step 7: Execute Linux Commands Using Ansible

Now let's execute Linux commands on both Managed Nodes without connecting to them manually.

### A. Check Hostname

Execute:

```bash
ansible managed -i inventory.ini -m command -a "hostname"
```

**Explanation:**

- `managed`: Executes the command on all servers in the managed group.
- `-m command`: Uses Ansible's command module.
- `-a "hostname"`: Executes the Linux hostname command.

### Actual result

Both servers successfully returned their hostnames.

```text
node1 | CHANGED | rc=0 >>
ip-172-31-31-231

node2 | CHANGED | rc=0 >>
ip-172-31-26-245
```

**What did we achieve?**

We retrieved the hostnames of both Managed Nodes from the Control Node without logging into each server manually.

### B. Check System Uptime

Execute:

```bash
ansible managed -i inventory.ini -m command -a "uptime"
```

**Explanation:**

- `uptime`: Displays how long the system has been running.
- It also displays the number of logged-in users and system load averages.

### Actual result

Both Managed Nodes successfully returned their uptime information.

```text
node1 | CHANGED | rc=0 >>
08:35:53 up 1:48, 1 user, load average: 0.10, 0.03, 0.01

node2 | CHANGED | rc=0 >>
08:35:53 up 1:48, 1 user, load average: 0.28, 0.11, 0.04
```

The uptime and load averages will naturally differ depending on when the command is executed.

**What did we achieve?**

We retrieved system uptime and load averages from both servers using a single Ansible command.

## 12. Understanding the Python Interpreter Warning

During execution, we received a warning similar to:

```text
[WARNING]: Host is using the discovered Python interpreter
at '/usr/bin/python3.14'
```

### Why did this happen?

Ansible automatically discovered Python 3.14 on the Managed Nodes.

It successfully used that Python interpreter to execute Ansible modules.

This is an interpreter discovery warning, not an execution error.

Our commands still executed successfully.

## 13. Important Ansible Commands

| Command | Purpose |
|---|---|
| `mkdir ansible-practicals` | Creates project directory |
| `cd ansible-practicals` | Navigates into project directory |
| `touch inventory.ini` | Creates inventory file |
| `vim inventory.ini` | Edits inventory file |
| `cat inventory.ini` | Displays inventory contents |
| `ansible-inventory -i inventory.ini --list` | Displays inventory in JSON |
| `ansible-inventory -i inventory.ini --graph` | Displays inventory as a tree |
| `ansible managed -i inventory.ini -m ping` | Tests all Managed Nodes |
| `ansible node1 -i inventory.ini -m ping` | Tests Managed Node 1 |
| `ansible node2 -i inventory.ini -m ping` | Tests Managed Node 2 |
| `ansible managed -i inventory.ini -m command -a "hostname"` | Checks hostnames |
| `ansible managed -i inventory.ini -m command -a "uptime"` | Checks system uptime |

## 14. Final Result

Successfully configured an Ansible inventory file and added two Managed Nodes.

### Ansible Ping Test

**Status: PASSED**

| Managed Node | Result |
|---|---|
| Node 1 | SUCCESS – pong |
| Node 2 | SUCCESS – pong |

Both Managed Nodes successfully responded to Ansible's ping module.

### Hostname Command

**Status: PASSED**

Both servers returned their hostnames:

- `ip-172-31-31-231`
- `ip-172-31-26-245`

### Uptime Command

**Status: PASSED**

Both servers returned their uptime and load averages.

### Overall Achievement

- Created an Ansible project directory.
- Created and configured an inventory file.
- Added two Managed Nodes.
- Configured SSH connection details.
- Verified inventory using `ansible-inventory`.
- Successfully tested connectivity using the Ansible ping module.
- Executed Linux commands remotely using Ansible.

**Practical 3 Completed: Ansible Inventory Configuration.**

## 15. Interview-Ready Explanation

In this practical, I created an Ansible inventory file on the Control Node and added two Managed Nodes using their private IP addresses.

I configured the remote username and SSH private key path in the inventory file.

Then I verified the inventory using `ansible-inventory` and tested connectivity using Ansible's ping module.

Both Managed Nodes returned `pong` successfully.

Finally, I executed Linux commands such as `hostname` and `uptime` on both servers using Ansible ad-hoc commands without manually connecting to each server.

This helped me understand how Ansible uses an inventory file to manage multiple remote servers from a single Control Node.

**Conclusion:** We successfully configured an Ansible inventory and established communication with two Managed Nodes. We also executed remote Linux commands from a single Control Node using Ansible.
