Day 3 – Introduction to Ansible Playbooks

Today I learned about Ansible Playbooks and how to use them to automate tasks on remote servers.

Before creating our first Playbook, I learned the basics of YAML, which is the format used to write Ansible Playbooks.

The main topics covered today are:

1. YAML Basics
2. Ansible Playbook Structure
3. Plays, Tasks and Modules
4. Ansible Collections
5. Creating Our First Ansible Playbook
6. Deploying a Static Website on AWS EC2

---

1. What is YAML?

YAML stands for YAML Ain't Markup Language.

It is a human-readable data serialization language used to store and exchange structured data.

In simple words, YAML helps us organize data in a clean and readable format.

YAML is widely used in DevOps tools such as:

- Ansible
- Kubernetes
- Docker Compose
- GitHub Actions

Why Do We Use YAML?

Imagine we have a Python application that needs some information as input.

We can provide that information using different formats:

- Text files
- JSON
- YAML

The problem with normal text files is that they don't have a standard structure.

Different people may organize the same information differently, making it difficult for applications to understand.

JSON and YAML solve this problem by providing a standard structure for representing data.

Why YAML instead of JSON?

YAML is easier for humans to read and write because:

- It uses simple syntax.
- It does not require curly braces for dictionaries.
- It does not require commas between list items.
- It uses indentation to organize data.

That's why Ansible uses YAML for writing Playbooks.

---

2. YAML Basics

To understand YAML, we need to learn four basic concepts:

1. Strings, Numbers and Booleans
2. Lists
3. Dictionaries
4. List of Dictionaries

A. Strings, Numbers and Booleans

Let's create a YAML file.

vim yuvraj.yaml

Start every YAML document with three hyphens:

---

Now let's write some information:

---
name: Yuvi
age: 25
working: true

Explanation:

Field| Data Type| Value
name| String| Yuvi
age| Integer| 25
working| Boolean| true

Remember:

- String represents text.
- Integer represents numbers.
- Boolean represents "true" or "false".

Strings can also be written using single or double quotes.

Example:

name: "Yuvraj"
city: 'Bengaluru'

B. Lists

A List is used when we have multiple values under one variable.

Example: Imagine I have multiple tasks to complete.

---
name: Yuvraj

tasks:
  - Learn Ansible
  - Practise Linux
  - Learn Docker
  - Work on DevOps projects

Explanation:

- "tasks" is the list name.
- Each item starts with a hyphen followed by a space.
- All list items have the same indentation.

Remember: Use "-" to define each item in a list.

C. Dictionaries

A Dictionary contains data in key-value pairs.

Example: Imagine I want to store my address.

---
address:
  street: MG Road
  city: Bengaluru
  state: Karnataka
  country: India

Explanation:

Key| Value
street| MG Road
city| Bengaluru
state| Karnataka
country| India

Here, "address" is the dictionary and each item inside it is a key-value pair.

Remember: A dictionary uses key-value pairs and does not require hyphens for each item.

D. List of Dictionaries

A List of Dictionaries combines both concepts.

Example: Imagine we have information about two students.

---
students:
  - name: Roman
    age: 22
    city: Mysuru

  - name: Seth
    age: 23
    city: Bengaluru

Here:

- "students" is a list.
- Each student is a dictionary.
- Each dictionary contains name, age and city.

YAML Quick Revision

Concept| Purpose| Syntax
String| Stores text| name: Yuvraj
Number| Stores numbers| "age: 25"
Boolean| Stores true/false| "working: true"
List| Stores multiple values| "- item"
Dictionary| Stores key-value pairs| "key: value"
List of Dictionaries| Stores multiple structured records| List with key-value pairs

Important YAML Rules:

1. Start the YAML document with "---".
2. Use spaces for indentation.
3. Never use tabs for indentation.
4. Use a colon followed by a space between keys and values.
5. Use hyphens for list items.
6. Maintain consistent indentation.

---

3. What is an Ansible Playbook?

Ansible Playbook is a YAML file that contains instructions for Ansible to execute.

In simple words, a Playbook tells Ansible:

- Which servers to target.
- What tasks to perform.
- Which modules to use.
- What actions should be executed.

For example, if we want to install Apache on multiple servers, we can write a Playbook instead of executing commands manually.

Playbook Structure

An Ansible Playbook is a list of Plays.

A Playbook can contain one or multiple Plays depending on the requirement.

Example:

Imagine we have two requirements:

1. Install and configure a Database Server.
2. Install and configure an Application Server.

We can create two separate Plays inside one Playbook.

---
- name: Configure Database Server
  hosts: db
  tasks:
    - name: Install database package
      apt:
        name: postgresql
        state: present

- name: Configure Application Server
  hosts: app
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

Here:

- First Play manages Database Servers.
- Second Play manages Application Servers.

Each Play has its own tasks.

What Goes Inside a Play?

A Play mainly contains:

1. Name
2. Hosts
3. Remote User (optional)
4. Become
5. Tasks

Let's understand each one.

1. Name

Defines the name of the Play.

Example:

name: Install Apache

2. Hosts

Specifies which Managed Nodes should execute the Play.

Example:

hosts: all

Other examples:

hosts: app

hosts: db

3. Remote User

Specifies which user Ansible should use to connect to the Managed Nodes.

Example:

remote_user: ubuntu

We can also define the connection user in the inventory file using "ansible_user".

4. Become

Used to execute tasks with elevated privileges, usually through sudo.

Example:

become: true

This is useful when installing packages or managing system services.

5. Tasks

Tasks are the actions that Ansible needs to perform.

Example:

tasks:
  - name: Install Apache
    apt:
      name: apache2
      state: present

Here, the task installs Apache using the "apt" module.

---

4. What are Ansible Tasks?

A Task represents one action that Ansible needs to perform on a Managed Node.

Examples:

- Installing Apache.
- Creating a file.
- Copying a file.
- Starting a service.
- Creating a user.

A Play can contain multiple tasks.

Example:

---
- name: Configure Web Server
  hosts: all
  become: true

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache
      service:
        name: apache2
        state: started

Here we have two tasks:

1. Install Apache.
2. Start Apache.

Ansible executes these tasks in order.

---

5. What are Ansible Modules?

A Module is a small piece of code that performs a specific task on Managed Nodes.

In simple words, modules do the actual work in Ansible.

For example:

Module| Purpose
"apt"| Install or remove packages on Debian-based systems
"copy"| Copy files to remote servers
"file"| Create or manage files and directories
"service"| Manage services
"shell"| Execute shell commands
"command"| Execute commands
"user"| Create or manage users

Example

If our requirement is to install Apache, we use the "apt" module.

- name: Install Apache
  apt:
    name: apache2
    state: present

If our requirement is to copy an HTML file to a remote server, we use the "copy" module.

- name: Copy HTML file
  copy:
    src: index.html
    dest: /var/www/html/index.html

Where Can We Find Ansible Modules?

We can find the available modules in the official Ansible documentation.

Website:

"https://docs.ansible.com/ansible/latest/collections/ansible/builtin/" (https://docs.ansible.com/ansible/latest/collections/ansible/builtin/)

We don't need to memorize every module.

Whenever we have a requirement, we can search for the appropriate module and refer to its documentation for the correct syntax.

---

6. What are Ansible Collections?

Ansible Collections are packages that contain reusable Ansible content.

They can contain:

- Modules
- Plugins
- Roles
- Documentation

Collections help us organize and reuse automation content.

What is Ansible Builtin?

Ansible provides a collection called "ansible.builtin".

It contains many commonly used modules.

Examples:

- "ansible.builtin.apt"
- "ansible.builtin.copy"
- "ansible.builtin.file"
- "ansible.builtin.service"
- "ansible.builtin.ping"

For example:

- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present

Here, "ansible.builtin.apt" refers to the APT module from the built-in collection.

For beginners, we can also write "apt" instead of "ansible.builtin.apt".

Both refer to the same module in this example.

Remember:

- Module performs a particular task.
- Collection is a package containing modules and other reusable Ansible content.

---

7. Practical – Create Our First Ansible Playbook

Now let's create our first practical project.

Project: Install Apache and Deploy a Static Website on AWS EC2

Objective:

- Install Apache on an Ubuntu EC2 instance.
- Copy an HTML file to the Apache web server directory.
- Access the webpage using the EC2 public IP address.

Architecture

       CONTROL NODE
      (Ansible Installed)
              |
       Ansible Playbook
              |
             SSH
              |
       AWS EC2 INSTANCE
          (Ubuntu)
              |
       Install Apache
              |
       Copy index.html
              |
       Apache Web Server
              |
       Access Website
        Using Public IP

Step 1: Prepare the Project Directory

Create a project directory:

mkdir ansible-webserver
cd ansible-webserver

Create the following files:

ansible-webserver/
│
├── inventory.yaml
├── first-playbook.yaml
└── index.html

Step 2: Create the Inventory File

Create "inventory.yaml":

all:
  hosts:
    webserver:
      ansible_host: YOUR_EC2_PUBLIC_IP
      ansible_user: ubuntu
      ansible_ssh_private_key_file: /path/to/aws-key.pem

Replace:

- "YOUR_EC2_PUBLIC_IP" with your EC2 public IP.
- "/path/to/aws-key.pem" with the actual path to your PEM file.

Here, we have one Managed Node called "webserver".

Step 3: Create an HTML File

Create "index.html":

<!DOCTYPE html>
<html>
<head>
    <title>My First Ansible Project</title>
</head>
<body>
    <h1>Hello from Ansible!</h1>
    <p>This website was deployed using Ansible.</p>
</body>
</html>

This is the static webpage we are going to deploy.

Step 4: Create the Ansible Playbook

Create "first-playbook.yaml":

---
- name: Install Apache and Deploy Website
  hosts: all
  become: true

  tasks:

    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Copy HTML file
      copy:
        src: index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: true

Step 5: Understand the Playbook

Let's understand what each part does.

Play name:

name: Install Apache and Deploy Website

This is the name of our Play.

Hosts:

hosts: all

This tells Ansible to execute the Play on all servers in our inventory.

Become:

become: true

This allows Ansible to execute tasks with elevated privileges.

Task 1 – Install Apache

- name: Install Apache
  apt:
    name: apache2
    state: present

This installs Apache on the Ubuntu EC2 instance.

Task 2 – Copy HTML File

- name: Copy HTML file
  copy:
    src: index.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: '0644'

Explanation:

- "src" – Source file on the Control Node.
- "dest" – Destination location on the Managed Node.
- "owner" – File owner.
- "group" – File group.
- "mode" – File permissions.

The HTML file is copied from our project directory to the Apache web server directory.

Task 3 – Start Apache

- name: Start Apache
  service:
    name: apache2
    state: started
    enabled: true

Explanation:

- "service" – Module used to manage services.
- "name" – Name of the service.
- "state: started" – Makes sure Apache is running.
- "enabled: true" – Makes sure Apache starts automatically when the system boots.

Step 6: Execute the Playbook

Run the following command from the project directory:

ansible-playbook -i inventory.yaml first-playbook.yaml

Explanation:

- "ansible-playbook" – Command used to execute Playbooks.
- "-i inventory.yaml" – Specifies our inventory file.
- "first-playbook.yaml" – Name of our Playbook.

What Happens Internally?

When we execute the Playbook:

1. Ansible connects to the EC2 instance.
2. It gathers information about the Managed Node (Gathering Facts).
3. It installs Apache.
4. It copies the HTML file to "/var/www/html/".
5. It starts Apache and enables it at boot.
6. Ansible displays the execution results.

Step 7: Allow HTTP Traffic in AWS

Our Apache server may be running, but we cannot access the website from a browser unless HTTP traffic is allowed.

Go to:

AWS Console → EC2 → Security Groups → Inbound Rules → Edit Inbound Rules.

Add this rule:

Type| Protocol| Port| Source
HTTP| TCP| 80| 0.0.0.0/0

Save the rules.

This allows HTTP traffic to reach the EC2 instance.

For a public demo website, this is fine. In production, restrict access according to your requirements.

Step 8: Access the Website

Copy the Public IPv4 address of your EC2 instance.

Open your browser:

http://YOUR_EC2_PUBLIC_IP

You should see:

Hello from Ansible!

This means we successfully installed Apache and deployed our static website using an Ansible Playbook.

---

8. Important Ansible Commands – Day 3

Command| Purpose
"ansible --version"| Check Ansible version
"ansible all -m ping"| Test connectivity
"ansible-playbook -i inventory.yaml first-playbook.yaml"| Execute a Playbook
"ansible-playbook --syntax-check first-playbook.yaml"| Check Playbook syntax
"ansible-playbook -i inventory.yaml first-playbook.yaml -v"| Execute with verbose output

---

9. Day 3 – Quick Revision

Concept| Meaning
YAML| Human-readable data serialization language
Playbook| YAML file containing automation instructions
Play| A section inside a Playbook that targets hosts
Hosts| Managed Nodes where tasks are executed
Tasks| Actions that Ansible needs to perform
Modules| Components that perform the actual tasks
Collections| Packages containing reusable Ansible content
"ansible.builtin"| Ansible's built-in collection
"apt"| Package installation module
"copy"| Copies files to remote servers
"service"| Manages services
"become: true"| Enables privilege escalation
"state: present"| Ensures a package is installed
"state: started"| Ensures a service is running
"ansible-playbook"| Command used to execute Playbooks

---

What I Learned Today

Today I started learning Ansible Playbooks. Before that, I understood the basics of YAML, including strings, numbers, booleans, lists and dictionaries.

I learned that an Ansible Playbook is a YAML file containing instructions for automation. I also understood its structure and the difference between Plays, Tasks, Modules and Collections.

For practical learning, I created my first Ansible Playbook. I used it to install Apache on an AWS EC2 instance, copy a static HTML file and start the Apache service.

Finally, I accessed the deployed website using the EC2 public IP address.

Key Takeaway: Ansible Playbooks help us automate multiple tasks using simple YAML files. Instead of executing commands manually, we can write instructions once and let Ansible perform the tasks on our Managed Nodes.
