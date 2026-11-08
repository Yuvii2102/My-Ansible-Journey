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

I used three Ubuntu EC2 instances.

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

Basic syntax:

```bash
ansible <host-pattern> -i <inventory-file> -m <module> -a "<arguments>"
```

## 4. Step 1: Navigate to the Ansible Project

```bash
cd ~/ansible-practicals
```

Check the available files:

```bash
ls
```

My inventory file was already created in this directory.

## 5. Step 2: Check Connectivity

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

```bash
ansible managed -i inventory.ini -m command -a "hostname"
```

Both servers returned their hostnames:

```text
node1 | CHANGED | rc=0 >>
ip-172-31-31-231

node2 | CHANGED | rc=0 >>
ip-172-31-26-245
```

**What I learned:** I can retrieve information from multiple servers using a single Ansible command.

## 7. Step 4: Check System Uptime

```bash
ansible managed -i inventory.ini -m command -a "uptime"
```

This command displays:

- How long the system has been running.
- Number of logged-in users.
- System load averages.

**What I learned:** I can check the uptime and system load of multiple servers without connecting to them individually.

## 8. Step 5: Check Disk Space

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

I used the Ansible `file` module.

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo state=directory"
```

Explanation:

- `-m file`: Uses the Ansible file module.
- `path`: Specifies the directory location.
- `state=directory`: Creates a directory if it does not exist.

To verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -ld /home/ubuntu/ansible-demo"
```

The directory was successfully created on both Managed Nodes.

## 11. Step 8: Create a File on Both Managed Nodes

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo/test.txt state=touch"
```

To verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -l /home/ubuntu/ansible-demo/"
```

The `test.txt` file was created on both servers.

## 12. Step 9: Copy a File to Both Managed Nodes

First, I created a file on my Control Node.

```bash
echo "Hello from Ansible Control Node" > sample.txt
```

Then I used the Ansible copy module:

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

I installed the `tree` package on both Managed Nodes.

```bash
ansible managed -i inventory.ini -m apt -a "name=tree state=present update_cache=yes" --become
```

Explanation:

- `-m apt`: Uses the Ansible apt module.
- `name=tree`: Specifies the package name.
- `state=present`: Ensures the package is installed.
- `update_cache=yes`: Updates the APT package cache.
- `--become`: Executes the task with elevated privileges using sudo.

To verify:

```bash
ansible managed -i inventory.ini -m command -a "tree --version"
```

The package was successfully installed on both Managed Nodes.

## 14. Step 11: Install Nginx on Both Managed Nodes

I installed Nginx using Ansible.

```bash
ansible managed -i inventory.ini -m apt -a "name=nginx state=present update_cache=yes" --become
```

This installed Nginx on both Managed Nodes.

## 15. Step 12: Start Nginx Service

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=started" --become
```

`state=started` ensures that the Nginx service is running.

## 16. Step 13: Check Nginx Service Status

```bash
ansible managed -i inventory.ini -m command -a "systemctl status nginx --no-pager"
```

Expected status:

```text
Active: active (running)
```

This confirmed that Nginx was running on both servers.

## 17. Step 14: Restart Nginx Service

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=restarted" --become
```

This restarted Nginx on both Managed Nodes.

## 18. Step 15: Stop Nginx Service

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=stopped" --become
```

This stopped Nginx on both servers.

## 19. Step 16: Enable Nginx at System Startup

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

---

# 22. Complete Command Reference

This section contains all the commands I used in this practical, arranged in execution order.

I can use this section whenever I want to repeat the practical from scratch.

### A. Navigate to the project directory

```bash
cd ~/ansible-practicals
ls
```

### B. Check connectivity

```bash
ansible managed -i inventory.ini -m ping
```

### C. Check server information

```bash
ansible managed -i inventory.ini -m command -a "hostname"

ansible managed -i inventory.ini -m command -a "uptime"

ansible managed -i inventory.ini -m command -a "df -h"

ansible managed -i inventory.ini -m command -a "free -h"
```

### D. Create a directory

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo state=directory"
```

Verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -ld /home/ubuntu/ansible-demo"
```

### E. Create a file

```bash
ansible managed -i inventory.ini -m file -a "path=/home/ubuntu/ansible-demo/test.txt state=touch"
```

Verify:

```bash
ansible managed -i inventory.ini -m command -a "ls -l /home/ubuntu/ansible-demo/"
```

### F. Create and copy a file

Create on Control Node:

```bash
echo "Hello from Ansible Control Node" > sample.txt
```

Copy to both Managed Nodes:

```bash
ansible managed -i inventory.ini -m copy -a "src=sample.txt dest=/home/ubuntu/ansible-demo/sample.txt"
```

Verify:

```bash
ansible managed -i inventory.ini -m command -a "cat /home/ubuntu/ansible-demo/sample.txt"
```

### G. Install tree

```bash
ansible managed -i inventory.ini -m apt -a "name=tree state=present update_cache=yes" --become
```

Verify:

```bash
ansible managed -i inventory.ini -m command -a "tree --version"
```

### H. Install Nginx

```bash
ansible managed -i inventory.ini -m apt -a "name=nginx state=present update_cache=yes" --become
```

### I. Start Nginx

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=started" --become
```

### J. Check Nginx status

```bash
ansible managed -i inventory.ini -m command -a "systemctl status nginx --no-pager"
```

### K. Restart Nginx

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=restarted" --become
```

### L. Stop Nginx

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=stopped" --become
```

### M. Enable and start Nginx

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=started enabled=yes" --become
```

---

# 23. Final Result

In this practical, I successfully performed different administrative tasks on both Managed Nodes using Ansible Ad-hoc commands.

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

## 24. What I Learned

- How to execute commands on multiple servers using Ansible.
- How to use different Ansible modules.
- How to create files and directories remotely.
- How to copy files between servers.
- How to install packages using Ansible.
- How to manage services remotely.
- How to use privilege escalation with `--become`.

## 25. Interview-Ready Explanation

In this practical, I learned how to use Ansible Ad-hoc commands to perform administrative tasks on multiple Linux servers.

I used modules such as ping, command, file, copy, apt and service.

I checked server information, created files and directories, copied files, installed packages and managed Nginx services on two Managed Nodes from a single Control Node.

This helped me understand how Ansible simplifies server management by executing tasks remotely without manually connecting to every server.

## 26. Official Documentation References

I referred to the official Ansible documentation to understand the syntax, available modules and how Ad-hoc commands work.

### Ansible Ad-hoc Commands

- [Introduction to Ad-hoc Commands](https://docs.ansible.com/projects/ansible/latest/command_guide/intro_adhoc.html)

This explains the basic syntax of Ad-hoc commands and how to execute tasks on managed nodes.

### Ansible Modules

- [Ping Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html) – Testing connectivity.
- [Command Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/command_module.html) – Executing Linux commands.
- [File Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/file_module.html) – Creating and managing files and directories.
- [Copy Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/copy_module.html) – Copying files to remote servers.
- [APT Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html) – Installing and managing Ubuntu packages.
- [Service Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_module.html) – Managing services.

### Additional Documentation

- [Ansible Inventory Guide](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html) – Understanding inventory files and managed hosts.
- [Introduction to Ansible Modules](https://docs.ansible.com/projects/ansible/latest/module_plugin_guide/modules_intro.html) – Understanding how Ansible modules work.

**Note:** The Ansible documentation explains the module syntax and usage. Commands such as `hostname`, `uptime`, `df -h`, `free -h`, and `systemctl` are standard Linux commands that I executed through Ansible's command module.

## 27. Next Steps

After learning Ansible Ad-hoc commands, I will move on to:

1. Ansible Playbooks.
2. Writing YAML playbooks.
3. Installing packages using playbooks.
4. Managing services using playbooks.
5. Automating multiple tasks in a single playbook.

**Conclusion:** I successfully used Ansible Ad-hoc commands to manage two remote Linux servers from a single Control Node. I also learned how different Ansible modules help automate everyday server administration tasks.
