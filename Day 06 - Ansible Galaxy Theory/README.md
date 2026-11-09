# Ansible Galaxy – Role Deep Dive

## 1. What is Ansible Galaxy?

Ansible Galaxy is a platform where developers and system administrators can share, discover, download, and publish Ansible Roles.

It acts as a marketplace for Ansible Roles.

Instead of creating every automation task from scratch, we can search for existing roles that have already been developed and published by other people.

For example, if I want to install and configure Docker on multiple servers, I can search for an existing Docker role on Ansible Galaxy and use it instead of writing all the tasks myself.

Website: https://galaxy.ansible.com/

## 2. Why do we use Ansible Galaxy?

Imagine that I need to install Docker on multiple managed nodes running different operating systems.

Some servers might use Ubuntu, others might use Debian or Red Hat.

The installation commands and configurations can differ between operating systems.

Writing all these conditions and tasks manually can make the playbook complicated.

Instead, I can search Ansible Galaxy for an existing Docker role that supports multiple operating systems.

This saves development time and avoids rewriting common automation tasks.

### Advantages of Ansible Galaxy

- **Reusability:** We can use existing roles instead of writing everything from scratch.
- **Time Saving:** We can reduce the time required to develop automation.
- **Community Sharing:** Developers can share roles with others.
- **Modularity:** Roles help keep automation organized.
- **Easy Maintenance:** We can use documented and maintained roles for common tasks.

## 3. Ansible Galaxy Website

Ansible Galaxy provides a platform to search for existing roles.

We can:

1. Search for roles based on our requirements.
2. Read role documentation.
3. Check supported operating systems.
4. View the author's GitHub repository.
5. Install and use the role in our playbooks.
6. Publish our own roles.

Before using a role, it is important to read its documentation and inspect its source code.

## 4. Ansible Galaxy CLI

Ansible Galaxy also provides a command-line utility called `ansible-galaxy`.

We can use this utility to manage Ansible Roles directly from the terminal.

Check the available commands:

```bash
ansible-galaxy -h
```

To see role-related commands:

```bash
ansible-galaxy role -h
```

Some important operations include:

| Command | Purpose |
|---|---|
| `init` | Creates a new Ansible Role |
| `install` | Installs an existing role |
| `remove` | Removes an installed role |
| `list` | Lists installed roles |
| `search` | Searches for roles |
| `import` | Imports a role from GitHub into Ansible Galaxy |

## 5. Installing Existing Roles

One of the main advantages of Ansible Galaxy is that we can use roles created by other developers.

For example, if I want to install Docker, I can search for a suitable Docker role on Ansible Galaxy.

After selecting a role, I can install it using:

```bash
ansible-galaxy role install AUTHOR.ROLE_NAME
```

The actual author and role name must be taken from the role's Ansible Galaxy page.

After installation, Ansible stores the role in its configured roles directory.

For a typical user installation, this may be:

```text
~/.ansible/roles/
```

We can check the installed roles using:

```bash
ansible-galaxy role list
```

## 6. Using an Installed Role in a Playbook

After installing a role, we can call it from an Ansible playbook.

Example:

```yaml
---
- name: Configure Docker
  hosts: all
  become: true

  roles:
    - AUTHOR.ROLE_NAME
```

Here:

- `hosts: all` – Runs the playbook against all hosts in the selected inventory.
- `become: true` – Uses privilege escalation.
- `roles` – Specifies the roles to execute.
- `AUTHOR.ROLE_NAME` – Refers to the installed role.

Ansible executes the tasks defined inside that role.

This allows us to use existing automation without writing every task manually.

## 7. Publishing Our Own Roles to Ansible Galaxy

Ansible Galaxy also allows developers to share their own roles.

For example, we created an Apache deployment role named `httpd` in our previous practical.

We can publish this role so that other developers can discover and use it.

The general process is:

1. Create an Ansible Role.
2. Push the role's source code to a GitHub repository.
3. Update the role documentation.
4. Configure the role metadata.
5. Sign in to Ansible Galaxy using GitHub.
6. Generate or obtain an Ansible Galaxy API token.
7. Import the GitHub repository into Ansible Galaxy.
8. Verify that the role appears in our Ansible Galaxy namespace.

### Why is GitHub required?

The role's source code is maintained in a GitHub repository.

Ansible Galaxy imports the role from that repository and makes it available through the Galaxy platform.

### Role Metadata

The `meta/main.yml` file contains information about the role.

It can include:

- Author
- Description
- Company
- License
- Supported platforms
- Role dependencies

This information helps other developers understand the purpose and requirements of the role.

### Role Documentation

The `README.md` file explains how to use the role.

It should include:

- Role description
- Requirements
- Supported platforms
- Variables
- Installation instructions
- Usage examples

Good documentation makes a role easier for other developers to understand and reuse.

## 8. Ansible Galaxy API Token

When importing a GitHub repository into Ansible Galaxy, authentication is required.

An API token is used to authenticate the import operation.

The token can be generated through the Ansible Galaxy account settings.

It must be handled securely and should never be committed to GitHub or shared publicly.

## 9. Ansible Galaxy Workflow

The overall workflow can be understood in two ways.

### Using Existing Roles

1. Search for a role on Ansible Galaxy.
2. Read its documentation.
3. Check its supported platforms.
4. Install the role using `ansible-galaxy`.
5. Reference the installed role in a playbook.
6. Execute the playbook.

### Publishing Our Own Roles

1. Create an Ansible Role.
2. Upload the role source code to GitHub.
3. Update `README.md` and `meta/main.yml`.
4. Sign in to Ansible Galaxy.
5. Obtain an API token.
6. Import the GitHub repository.
7. Verify the published role.

## 10. Key Takeaways

- Ansible Galaxy is a marketplace for Ansible Roles.
- `ansible-galaxy` is the CLI utility used to manage roles.
- We can install and reuse roles created by other developers.
- Installed roles can be called directly from playbooks.
- We can publish our own roles through GitHub and Ansible Galaxy.
- Documentation and metadata are important when sharing roles.

**Key Learning:** Ansible Galaxy helps developers reuse existing automation and share their own Ansible Roles with the community.
