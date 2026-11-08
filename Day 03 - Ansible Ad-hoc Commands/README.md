# Practical 4: Ansible Ad-hoc Commands

## 1. Objective

In this practical, I learned how to execute commands on multiple remote servers using Ansible Ad-hoc commands.

Instead of connecting to each Managed Node manually, I used the Ansible Control Node to execute commands on both servers.

I also learned how to:

- Check server information.
- Check disk and memory usage.
- Create files and directories.
- Copy files between servers.
- Install packages.
- Start, stop and restart services.

## 2. Lab Setup

I used three Ubuntu EC2 instances for this practical.

| Instance | Role | Private IP |
|---|---|---|
| EC2 Instance 1 | Control Node | 172.31.28.28 |
| EC2 Instance 2 | Managed Node 1 | 172.31.31.231 |
| EC2 Instance 3 | Managed Node 2 | 172.31.26.245 |

**Username:** ubuntu

I used the inventory file created in Practical 3 to manage both Managed Nodes.

## 3. What are Ansible Ad-hoc Commands?

Ansible Ad-hoc commands are used to execute simple tasks on remote servers without creating a playbook.

For example, if I want to check the hostname of two servers, I don't need to connect to each server manually.

I can execute a single command from the Control Node and Ansible will run it on both servers.

Example:

```bash
ansible managed -i inventory.ini -m command -a "hostname"
```

## 4. Step 1: Navigate to the Ansible Project

First, I connected to my Control Node and navigated to my Ansible project directory.

```bash
cd ~/ansible-practicals
```

I checked the available files:

```bash
ls
```

My inventory file was already created in this directory.

## 5. Step 2: Check Connectivity

Before executing commands, I checked whether both Managed Nodes were reachable through Ansible.

```bash
ansible managed -i inventory.ini -m ping
```

Both Managed Nodes returned:

```text
node1 | SUCCESS => {
    "ping": "pong"
}

node2 | SUCCESS => {
    "ping": "pong"
}
```

This confirmed that Ansible could communicate with both servers.

**What I learned:** The Ansible ping module checks Ansible connectivity and remote Python availability. It is different from the normal Linux ping command.

## 6. Step 3: Check Hostname

I used the `hostname` command to check the hostname of both Managed Nodes.

```bash
ansible managed -i inventory.ini -m command -a "hostname"
```

The servers returned:

```text
node1 | CHANGED | rc=0 >>
ip-172-31-31-231

node2 | CHANGED | rc=0 >>
ip-172-31-26-245
```

**What I learned:** I can retrieve information from multiple servers using a single Ansible command.

## 7. Step 4: Check System Uptime

Next, I checked how long both servers had been running.

```bash
ansible managed -i inventory.ini -m command -a "uptime"
```

This command displays:

- How long the system has been running.
- Number of logged-in users.
- System load averages.

**What I learned:** I can check the uptime and system load of multiple servers without connecting to them individually.

## 8. Step 5: Check Disk Space

I used the `df -h` command to check disk usage on both Managed Nodes.

```bash
ansible managed -i inventory.ini -m command -a "df -h"
```

The output displayed:

- Filesystem
- Size
- Used space
- Available space
- Usage percentage
- Mount location

**What I learned:** I can monitor disk space on multiple servers using Ansible.

## 9. Step 6: Check Memory Usage

I used the `free -h` command to check memory usage.

```bash
ansible managed -i inventory.ini -m command -a "free -h"
```

This command displays:

- Total RAM
- Used RAM
- Free RAM
- Shared memory
- Buffer and cache
- Available memory
- Swap memory

**What I learned:** I can check the memory status of multiple servers from the Control Node.

## 10. Step 7: Create a Directory on Both Managed Nodes

Next, I learned how to create directories using the Ansible `file` module.

I executed:

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo state=directory"
```

### Understanding the command

- `-m file`: Uses the Ansible file module.
- `path`: Specifies the directory location.
- `state=directory`: Creates a directory if it does not exist.

To verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -ld /home/ubuntu/ansible-demo"
```

The directory was successfully created on both Managed Nodes.

## 11. Step 8: Create a File on Both Managed Nodes

I used the Ansible file module to create a file inside the directory.

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo/test.txt state=touch"
```

To verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -l /home/ubuntu/ansible-demo/"
```

The `test.txt` file was created on both servers.

**What I learned:** The Ansible file module can create files and directories on remote servers.

## 12. Step 9: Copy a File to Both Managed Nodes

First, I created a file on my Control Node.

```bash
echo "Hello from Ansible Control Node" > sample.txt
```

Then I used the Ansible copy module to transfer the file to both Managed Nodes.

```bash
ansible managed -i inventory.ini -m copy -a "src=sample.txt dest=/home/ubuntu/ansible-demo/sample.txt"
```

To verify:

```bash
ansible managed -i inventory.ini -m command -a "cat /home/ubuntu/ansible-demo/sample.txt"
```

Expected output:

```text
Hello from Ansible Control Node
```

The file was successfully copied to both servers.

**What I learned:** The Ansible copy module helps transfer files from the Control Node to remote Managed Nodes.

## 13. Step 10: Install Packages Using Ansible

Next, I learned how to install packages on multiple servers using the Ansible `apt` module.

I installed the `tree` package on both Managed Nodes.

```bash
ansible managed -i inventory.ini -m apt -a "name=tree state=present update_cache=yes" --become
```

### Understanding the command

- `-m apt`: Uses the Ansible apt module.
- `name=tree`: Specifies the package name.
- `state=present`: Ensures the package is installed.
- `update_cache=yes`: Updates the APT package cache.
- `--become`: Executes the task with elevated privileges using sudo.

To verify the installation:

```bash
ansible managed -i inventory.ini -m command -a "tree --version"
```

The package was successfully installed on both Managed Nodes.

## 14. Step 11: Install Nginx on Both Managed Nodes

I also learned how to install Nginx using Ansible.

```bash
ansible managed -i inventory.ini -m apt -a "name=nginx state=present update_cache=yes" --become
```

This installed Nginx on both Managed Nodes.

## 15. Step 12: Start Nginx Service

After installing Nginx, I started the service using the Ansible service module.

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=started" --become
```

`state=started` ensures that the Nginx service is running.

## 16. Step 13: Check Nginx Service Status

I checked whether Nginx was running on both Managed Nodes.

```bash
ansible managed -i inventory.ini -m command -a "systemctl status nginx --no-pager"
```

Expected status:

```text
Active: active (running)
```

This confirmed that Nginx was running on both servers.

## 17. Step 14: Restart Nginx Service

I used the service module to restart Nginx.

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=restarted" --become
```

This restarted Nginx on both Managed Nodes.

## 18. Step 15: Stop Nginx Service

Next, I stopped the Nginx service.

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=stopped" --become
```

This stopped Nginx on both servers.

## 19. Step 16: Enable Nginx at System Startup

Finally, I enabled Nginx so that it starts automatically whenever the servers boot.

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=started enabled=yes" --become
```

Here:

- `state=started`: Ensures Nginx is running.
- `enabled=yes`: Enables Nginx to start automatically during system boot.

I left Nginx running and enabled on both Managed Nodes.

## 20. Important Ansible Modules Used

| Module | Purpose |
|---|---|
| `ping` | Tests Ansible connectivity |
| `command` | Executes Linux commands |
| `file` | Creates and manages files and directories |
| `copy` | Copies files to remote servers |
| `apt` | Installs and manages Ubuntu packages |
| `service` | Manages system services |

## 21. Understanding --become

During package installation and service management, I used:

```bash
--become
```

It allows Ansible to execute tasks with elevated privileges.

For example, installing Nginx requires administrator permissions.

Instead of manually logging in and using sudo on every server, Ansible uses privilege escalation to execute the task.

## 22. Final Result

In this practical, I successfully performed different administrative tasks on both Managed Nodes using Ansible Ad-hoc commands.

### Tasks Completed

| Task | Result |
|---|---|
| Check connectivity | Successful |
| Check hostname | Successful |
| Check system uptime | Successful |
| Check disk space | Successful |
| Check memory usage | Successful |
| Create directory | Successful |
| Create file | Successful |
| Copy file | Successful |
| Install tree package | Successful |
| Install Nginx | Successful |
| Start Nginx | Successful |
| Check Nginx status | Successful |
| Restart Nginx | Successful |
| Stop Nginx | Successful |
| Enable Nginx at startup | Successful |

### What I Learned

- How to execute commands on multiple servers using Ansible.
- How to use different Ansible modules.
- How to create files and directories remotely.
- How to copy files between servers.
- How to install packages using Ansible.
- How to manage services remotely.
- How to use privilege escalation with `--become`.

## 23. Interview-Ready Explanation

In this practical, I learned how to use Ansible Ad-hoc commands to perform administrative tasks on multiple Linux servers.

I used modules such as ping, command, file, copy, apt and service.

I checked server information, created files and directories, copied files, installed packages and managed Nginx services on two Managed Nodes from a single Control Node.

This helped me understand how Ansible simplifies server management by executing tasks remotely without manually connecting to every server.


**Conclusion:** I successfully used Ansible Ad-hoc commands to manage two remote Linux servers from a single Control Node. I also learned how different Ansible modules help automate everyday server administration tasks.
