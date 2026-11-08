# Practical – Deploy Apache and Host a Static Website Using Ansible Playbook

## 1. Objective

In this practical, I used an Ansible Playbook to automate the installation and configuration of an Apache web server on two AWS EC2 managed nodes.

I also created a simple HTML website and deployed it automatically using Ansible.

## 2. Lab Architecture

I used three Ubuntu EC2 instances:

| Instance | Role | Private IP |
|---|---|---|
| EC2 Instance 1 | Ansible Control Node | 172.31.28.28 |
| EC2 Instance 2 | Managed Node 1 | 172.31.31.231 |
| EC2 Instance 3 | Managed Node 2 | 172.31.26.245 |

**Architecture flow:**

Control Node → SSH → Managed Nodes → Install Apache → Deploy HTML Website

## 3. Prerequisites

Before starting this practical, I completed the following:

- Installed Ansible on the Control Node.
- Created two Ubuntu managed nodes.
- Configured SSH key-based authentication.
- Created an Ansible inventory file.
- Practiced Ansible ad-hoc commands.
- Verified connectivity using the Ansible ping module.

## 4. Project Structure

I created the following files inside my Ansible project directory:

```text
ansible-practicals/
│
├── inventory.ini
├── index.html
└── first-playbook.yaml
```

I worked from the Control Node.

```bash
cd ~/ansible-practicals
```

## 5. Step 1 – Create an HTML File

First, I created a simple HTML file that would be hosted on the Apache web server.

```bash
nano index.html
```

I added the following HTML code:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First Ansible Website</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <h2>My First Ansible Playbook</h2>
    <p>This website was deployed automatically using Ansible.</p>
</body>
</html>
```

I saved the file.

**Purpose:** This file contains the static website that I want to deploy on both managed nodes.

## 6. Step 2 – Create the Ansible Playbook

Next, I created a YAML file.

```bash
nano first-playbook.yaml
```

I added the following playbook:

```yaml
---
- name: Install Apache and deploy website
  hosts: managed
  become: true

  tasks:
    - name: Install Apache web server
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Copy HTML file to Apache web directory
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Start Apache service
      ansible.builtin.service:
        name: apache2
        state: started
        enabled: yes
```

### Understanding the playbook

**1. hosts: managed**

Specifies that the playbook should execute on all hosts belonging to the `managed` group in my inventory file.

**2. become: true**

Enables privilege escalation using sudo. This is required for administrative operations such as installing Apache and copying files into protected directories.

**3. tasks**

Contains the operations that Ansible should execute.

### Task 1 – Install Apache

```yaml
ansible.builtin.apt:
  name: apache2
  state: present
  update_cache: yes
```

- `apt`: Package management module.
- `name`: Specifies the Apache package.
- `state: present`: Ensures Apache is installed.
- `update_cache: yes`: Updates the APT package cache before installation.

### Task 2 – Copy the HTML file

```yaml
ansible.builtin.copy:
  src: index.html
  dest: /var/www/html/index.html
  owner: root
  group: root
  mode: '0644'
```

- `src`: Source HTML file on the Control Node.
- `dest`: Destination path on the managed nodes.
- `owner`: Sets the file owner to root.
- `group`: Sets the file group to root.
- `mode`: Sets file permissions to 0644.

### Task 3 – Start Apache

```yaml
ansible.builtin.service:
  name: apache2
  state: started
  enabled: yes
```

- `service`: Service management module.
- `name`: Apache service.
- `state: started`: Ensures Apache is running.
- `enabled: yes`: Ensures Apache starts automatically during system boot.

## 7. Step 3 – Check Playbook Syntax

Before executing the playbook, I checked whether the YAML syntax was valid.

```bash
ansible-playbook -i inventory.ini first-playbook.yaml --syntax-check
```

**Expected result:**

```text
playbook: first-playbook.yaml
```

This confirms that Ansible can parse the playbook successfully.

## 8. Step 4 – Execute the Playbook

I executed the playbook using:

```bash
ansible-playbook -i inventory.ini first-playbook.yaml
```

Ansible performed the following operations:

1. Connected to both managed nodes through SSH.
2. Gathered system facts.
3. Installed Apache.
4. Copied the HTML file.
5. Started and enabled the Apache service.

The tasks were executed on both managed nodes.

## 9. Step 5 – Verify Apache Service

After executing the playbook, I checked whether Apache was running on both nodes.

```bash
ansible managed -i inventory.ini -m command -a "systemctl is-active apache2"
```

**Actual result:**

```text
node1 | CHANGED | rc=0 >>
active

node2 | CHANGED | rc=0 >>
active
```

Both Apache services were active.

## 10. Step 6 – Verify Port 80

I checked whether Apache was listening on HTTP port 80.

```bash
ansible managed -i inventory.ini -m command -a "ss -lntp"
```

I confirmed that port 80 was in the LISTEN state on both managed nodes.

## 11. Step 7 – Verify the Website Locally

I tested the Apache HTTP response from the Control Node.

```bash
ansible managed -i inventory.ini -m command -a "curl -I http://localhost"
```

**Actual result:**

```text
HTTP/1.1 200 OK
Server: Apache/2.4.66 (Ubuntu)
Content-Type: text/html
```

Both managed nodes returned HTTP 200 OK.

This confirmed that Apache was serving HTTP responses successfully.

## 12. Step 8 – Configure AWS Security Group

To allow external HTTP access, I checked the Security Group attached to the managed instances.

I verified these inbound rules:

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | Control Node / authorized source |
| HTTP | TCP | 80 | My IP or 0.0.0.0/0 for a temporary public demo |

I also verified the VPC route table, Internet Gateway, subnet public IPv4 configuration and Network ACL.

## 13. Step 9 – Test Using Public IP Addresses

I obtained the current public IPv4 addresses of both managed nodes from the AWS EC2 console.

I tested the HTTP response from the Control Node:

```bash
curl -I --connect-timeout 5 http://NODE1_PUBLIC_IP
curl -I --connect-timeout 5 http://NODE2_PUBLIC_IP
```

Both returned:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.66 (Ubuntu)
```

## 14. Step 10 – Verify From My Windows Computer

I opened Windows PowerShell and tested connectivity to both managed nodes.

```powershell
Test-NetConnection NODE1_PUBLIC_IP -Port 80
```

```powershell
Test-NetConnection NODE2_PUBLIC_IP -Port 80
```

Both returned:

```text
TcpTestSucceeded : True
```

Next, I tested the actual HTTP response using curl:

```powershell
curl.exe -v http://NODE1_PUBLIC_IP
```

**Actual result:**

```text
HTTP/1.1 200 OK
Server: Apache/2.4.66 (Ubuntu)

<h1>Hello from Ansible!</h1>
<h2>My First Ansible Playbook</h2>
<p>This website was deployed automatically using Ansible.</p>
```

This confirmed that my Windows computer could access the deployed HTML content through HTTP.

Note: The browser continued to show a connection timeout even though PowerShell successfully received HTTP 200 OK. I left the browser-specific issue for separate troubleshooting.

## 15. Troubleshooting

### Issue 1 – Apache service failed to start

**Error:**

```text
Unable to start service apache2
```

**Cause:**

Nginx was already installed and running on both managed nodes from my previous practical. Both Nginx and Apache use port 80 by default.

**Solution:**

I stopped Nginx on both managed nodes:

```bash
ansible managed -i inventory.ini -m service -a "name=nginx state=stopped" --become
```

Then I executed the playbook again:

```bash
ansible-playbook -i inventory.ini first-playbook.yaml
```

Apache started successfully.

### Issue 2 – SSH connection timed out

**Error:**

```text
ssh: connect to host port 22: Connection timed out
```

**Cause:**

The SSH inbound rule was missing from the Security Group associated with the managed nodes.

**Solution:**

I verified the Security Group and ensured SSH access on TCP port 22 was allowed from the Control Node's authorized source.

After restoring SSH connectivity, Ansible could communicate with both managed nodes again.

### Issue 3 – Browser connection timed out

**Observation:**

The browser displayed:

```text
This site can't be reached
ERR_CONNECTION_TIMED_OUT
```

However, PowerShell successfully received:

```text
HTTP/1.1 200 OK
```

**Troubleshooting performed:**

- Verified Apache service status.
- Verified port 80 was listening.
- Checked Security Group inbound rules.
- Checked the VPC route table and Internet Gateway.
- Checked subnet public IPv4 settings.
- Checked Network ACL inbound and outbound rules.
- Tested both public IP addresses using curl.
- Tested TCP port 80 using PowerShell.

**Result:**

Both managed nodes were reachable from PowerShell and returned the deployed HTML content. The browser-specific timeout remained unresolved.

## 16. What I Learned

Through this practical, I learned:

1. How to create an Ansible Playbook using YAML.
2. How to define hosts and tasks in a playbook.
3. How to use built-in Ansible modules.
4. How to install Apache automatically using the APT module.
5. How to copy files from the Control Node to managed nodes.
6. How to manage services using the service module.
7. How to execute a playbook on multiple managed nodes.
8. How to verify Apache service status and HTTP responses.
9. How to troubleshoot port conflicts and SSH connectivity.
10. How to verify public HTTP connectivity using curl and PowerShell.

## 17. Important Commands Reference

| Command | Purpose |
|---|---|
| `nano index.html` | Create an HTML file |
| `nano first-playbook.yaml` | Create an Ansible Playbook |
| `ansible-playbook -i inventory.ini first-playbook.yaml --syntax-check` | Validate playbook syntax |
| `ansible-playbook -i inventory.ini first-playbook.yaml` | Execute the playbook |
| `ansible managed -i inventory.ini -m command -a "systemctl is-active apache2"` | Check Apache status |
| `ansible managed -i inventory.ini -m command -a "ss -lntp"` | Check listening ports |
| `ansible managed -i inventory.ini -m command -a "curl -I http://localhost"` | Test local HTTP response |
| `curl -I http://PUBLIC_IP` | Test HTTP from the Control Node |
| `Test-NetConnection PUBLIC_IP -Port 80` | Test TCP connectivity from Windows |
| `curl.exe -v http://PUBLIC_IP` | Inspect the HTTP request and response |

## 18. Official Documentation

- Ansible Playbooks: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html
- APT Module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html
- Copy Module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/copy_module.html
- Service Module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_module.html
- AWS Security Groups: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

---

## Final Result

I successfully created and executed my first Ansible Playbook.

Using one playbook, I installed Apache, deployed an HTML website and started the Apache service on two Ubuntu EC2 managed nodes.

Both instances returned HTTP 200 OK, and I verified that the deployed HTML content was accessible from my Windows computer using PowerShell.
