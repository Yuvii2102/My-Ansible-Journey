# Day 04 – Ansible Playbooks (Theory)

## 1. What is YAML?

YAML stands for **YAML Ain't Markup Language**.

It is a human-readable data serialization language used to store and organize data in a structured format.

Ansible uses YAML to write playbooks because it is easy to read and understand.

### Why YAML?

- Easy to read and write.
- Uses simple key-value pairs.
- Supports lists and dictionaries.
- Helps organize configuration data.
- Uses indentation to define structure.

### YAML vs JSON

Both YAML and JSON are used to represent structured data.

However, YAML is generally easier for humans to read and write because it has a simpler syntax.

## 2. YAML Basic Syntax

### A. Key-value pairs

A key stores a value.

```yaml
name: Yuvraj
age: 20
working: true
```

Common data types:

- String: `name: Yuvraj`
- Integer: `age: 20`
- Boolean: `working: true`

### B. Lists

A list stores multiple values under one key.

We use a hyphen (`-`) to define list items.

```yaml
skills:
  - Linux
  - Docker
  - AWS
  - Ansible
```

### C. Dictionaries

A dictionary stores related key-value pairs under one parent key.

```yaml
address:
  city: Mysuru
  state: Karnataka
  country: India
```

### D. List of Dictionaries

We can combine lists and dictionaries to store multiple structured records.

```yaml
students:
  - name: Rahul
    age: 21
    city: Bangalore

  - name: Arun
    age: 22
    city: Mysuru
```

### Important YAML rules

1. YAML files commonly start with `---`.
2. Use spaces for indentation. Do not use tabs.
3. Use a colon (`:`) between keys and values.
4. Use hyphens (`-`) for list items.
5. Maintain consistent indentation.

---

## 3. What is an Ansible Playbook?

An Ansible Playbook is a YAML file that contains instructions for automating tasks on managed nodes.

Instead of executing individual Ansible ad-hoc commands, we can write multiple tasks inside a playbook and execute them together.

**Example:** We can create a playbook to:

- Install Apache.
- Copy an HTML file.
- Start the Apache service.

### Advantages of Playbooks

- Automates multiple tasks.
- Reduces manual work.
- Makes configuration repeatable.
- Easy to read and maintain.
- Can manage multiple servers together.

## 4. Understanding Playbook Structure

An Ansible Playbook contains one or more plays.

Each play defines which hosts to manage and what tasks to execute.

A basic playbook structure:

```yaml
---
- name: Configure web servers
  hosts: managed
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: present
```

### Important components

**1. name**

Provides a description of the play or task.

**2. hosts**

Specifies which managed hosts should execute the play.

Example:

```yaml
hosts: managed
```

Here, `managed` refers to the group defined in our inventory file.

**3. become**

Enables privilege escalation, usually through sudo.

Example:

```yaml
become: true
```

It is similar to using `--become` in Ansible ad-hoc commands.

**4. tasks**

Contains the list of operations that Ansible should perform.

**5. modules**

Modules perform the actual operations defined inside tasks.

---

## 5. What are Ansible Modules?

Modules are small programs used by Ansible to perform specific operations on managed nodes.

They are responsible for carrying out the actual work.

Examples:

| Module | Purpose |
|---|---|
| `apt` | Installs or removes packages |
| `copy` | Copies files to managed nodes |
| `file` | Manages files and directories |
| `service` | Manages services |
| `ping` | Tests Ansible connectivity |

Example:

```yaml
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
```

Here:

- `ansible.builtin.apt` is the module.
- `name: apache2` specifies the package.
- `state: present` ensures Apache is installed.

We don't need to memorize every module. We can refer to the official Ansible documentation whenever required.

---

## 6. What are Ansible Collections?

Collections are packages that organize and distribute Ansible content.

A collection can contain:

- Modules
- Plugins
- Roles
- Documentation

Ansible provides built-in modules through the `ansible.builtin` collection.

Examples:

```yaml
ansible.builtin.apt
ansible.builtin.copy
ansible.builtin.file
ansible.builtin.service
```

Here, `ansible.builtin` identifies the collection, and the remaining part identifies the module.

### Why do we use Collections?

- Organize Ansible content.
- Group related modules and plugins.
- Make automation easier to manage.
- Allow users to install and use additional content.

---

## 7. How Ansible Playbooks Work

The general execution flow is:

1. We create a YAML playbook.
2. We define the target hosts.
3. We write tasks.
4. Each task uses an appropriate module.
5. Ansible connects to managed nodes through SSH.
6. Ansible gathers facts by default.
7. Ansible executes the tasks.
8. Ansible displays the execution results.

### Important commands

Check playbook syntax:

```bash
ansible-playbook -i inventory.ini first-playbook.yaml --syntax-check
```

Execute a playbook:

```bash
ansible-playbook -i inventory.ini first-playbook.yaml
```

---

## 8. Interview Questions and Answers

**Q1. What is YAML?**

YAML is a human-readable data serialization language used to represent structured data. Ansible uses YAML to write playbooks.

**Q2. What is an Ansible Playbook?**

An Ansible Playbook is a YAML file containing instructions to automate tasks on managed nodes.

**Q3. What is a Play in Ansible?**

A play defines the target hosts and the tasks that Ansible should execute on those hosts.

**Q4. What is a Task?**

A task is an individual operation inside a playbook. It generally uses a module to perform an action.

**Q5. What is an Ansible Module?**

A module is a small program that performs a specific operation, such as installing packages, copying files or managing services.

**Q6. What is an Ansible Collection?**

A collection is a package that groups Ansible content such as modules, plugins, roles and documentation.

**Q7. What is `become: true`?**

It enables privilege escalation, usually using sudo, so Ansible can perform administrative operations.

**Q8. What is the difference between an Ad-hoc command and a Playbook?**

An ad-hoc command performs an individual task directly from the command line, whereas a playbook contains multiple organized tasks written in YAML.

---

## 9. Quick Revision

| Term | Simple meaning |
|---|---|
| YAML | Language used to write playbooks |
| Playbook | File containing automation instructions |
| Play | Defines hosts and tasks |
| Hosts | Target managed nodes |
| Tasks | Operations to execute |
| Modules | Perform actual operations |
| Collections | Packages containing Ansible content |
| Become | Privilege escalation |

## 10. Official Documentation

- YAML documentation: https://yaml.org/
- Ansible Playbooks: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html
- Ansible Modules: https://docs.ansible.com/projects/ansible/latest/module_plugin_guide/modules_intro.html
- Ansible Collections: https://docs.ansible.com/projects/ansible/latest/collections_guide/index.html
- Built-in Modules: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/
