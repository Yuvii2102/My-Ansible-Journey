# Part 1: Theory of Ansible Roles

## 1. What are Ansible Roles?

Ansible Roles are a way of organizing Ansible playbooks into smaller, reusable, and structured components.

When we write all automation tasks inside a single playbook, the playbook can become large and difficult to manage.

Ansible Roles help us organize tasks, files, variables, templates, and handlers into separate directories.

This makes automation easier to maintain, reuse, and understand.

**Example:**

Instead of writing all Apache installation and configuration tasks inside one playbook, we can create a separate role called `httpd` and manage all Apache-related tasks inside it.

## 2. Why do we use Ansible Roles?

Ansible Roles provide several advantages:

- **Reusability:** We can reuse the same role in multiple playbooks.
- **Organization:** Tasks and configuration files are stored in separate directories.
- **Maintainability:** Changes can be made easily without modifying a large playbook.
- **Readability:** The playbook becomes smaller and easier to understand.
- **Scalability:** We can create separate roles for different services and applications.

## 3. Ansible Role Directory Structure

Ansible follows a standard directory structure when creating a role.

Example:

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

Each directory has a specific purpose.

### 3.1 defaults/

The `defaults/` directory contains default variables used by the role.

- These variables have the lowest precedence.
- They can be overridden by variables defined elsewhere.
- They are useful for providing default configuration values.

Example:

```yaml
---
apache_port: 80
```

### 3.2 files/

The `files/` directory stores static files that Ansible can copy to managed nodes.

Examples:

- HTML files
- Configuration files
- Scripts
- Other static application files

In our project, we store `index.html` inside this directory.

### 3.3 handlers/

The `handlers/` directory contains tasks that are triggered only when another task notifies them.

Handlers are commonly used for:

- Restarting services
- Reloading services
- Applying configuration changes

Example:

```yaml
---
- name: Restart Apache
  ansible.builtin.service:
    name: apache2
    state: restarted
```

A handler runs only when it is notified by a task.

### 3.4 meta/

The `meta/` directory contains information about the role.

It can define:

- Role dependencies
- Role metadata
- Author information
- Supported platforms

The `main.yml` file is used to define this information.

### 3.5 tasks/

The `tasks/` directory contains the main automation tasks of the role.

The `main.yml` file is the default entry point for these tasks.

For example, an Apache role can contain tasks to:

1. Install Apache.
2. Copy website files.
3. Start and enable the Apache service.

This is where most of the automation logic is written.

### 3.6 templates/

The `templates/` directory stores Jinja2 template files.

Templates allow us to create dynamic configuration files using variables.

For example, we can use a variable to dynamically configure a port number or server name.

Ansible processes template files using the `template` module.

### 3.7 tests/

The `tests/` directory contains files used for testing the role.

It commonly includes:

- `inventory` – Defines the test hosts.
- `test.yml` – A playbook used to test the role.

These files help verify whether the role works as expected.

### 3.8 vars/

The `vars/` directory contains variables used by the role.

- Variables defined here have higher precedence than variables in `defaults/`.
- They are useful for role-specific configuration.
- They are generally harder to override than default variables.

The `main.yml` file stores these variables.

### 3.9 README.md

The `README.md` file contains documentation about the role.

It can explain:

- What the role does.
- Its requirements.
- Variables used.
- How to execute the role.
- Expected results.

## 4. How Ansible Roles Work

Ansible Roles follow a structured execution process.

1. We create an Ansible role.
2. We define automation tasks inside the role's `tasks/main.yml` file.
3. We place required static files inside the `files/` directory.
4. We configure the role in an Ansible playbook.
5. Ansible executes the tasks defined inside the role.
6. The managed nodes receive the required configuration.

## 5. Roles vs Traditional Playbooks

| Traditional Playbook | Ansible Roles |
|---|---|
| Tasks are commonly written directly inside a playbook. | Tasks are organized into separate directories. |
| Can become difficult to manage as tasks increase. | Provides a structured approach. |
| Reusability requires additional organization. | Roles can be reused across playbooks. |
| Suitable for simple automation. | Suitable for modular and larger automation projects. |

## 6. How Roles Are Called in a Playbook

An Ansible playbook can execute a role using the `roles` section.

Example:

```yaml
---
- name: Deploy Website Using Ansible Roles
  hosts: all
  become: true

  roles:
    - httpd
```

Here:

- `hosts: all` – Runs the play against all hosts in the selected inventory.
- `become: true` – Executes tasks with elevated privileges.
- `roles:` – Defines the roles to execute.
- `httpd` – The name of the role that Ansible executes.

Ansible automatically looks for the role's default task entry point at `httpd/tasks/main.yml`.

## 7. Important Ansible Role Concepts

| Concept | Purpose |
|---|---|
| Role | Organizes automation into reusable components. |
| Tasks | Defines the operations to execute. |
| Files | Stores static files to be copied. |
| Handlers | Executes tasks when notified. |
| Variables | Stores configurable values. |
| Templates | Generates dynamic configuration files. |
| Metadata | Defines role information and dependencies. |
| Inventory | Defines the managed hosts. |
| Playbook | Connects hosts, privileges, and roles. |

## 8. Project Objective

The objective of this project is to understand how Ansible Roles help organize automation into reusable and maintainable components.

We use an Apache web server deployment as our practical example.

Instead of managing all deployment tasks in a single playbook, we organize the Apache installation, website file, and service management inside a dedicated role named `httpd`.

This approach makes our Ansible automation cleaner, more structured, and easier to maintain.

---

**End of Part 1 – Theory**

**Part 2 – Practical Implementation:** Creating an Ansible Role and deploying a static website using Apache.
