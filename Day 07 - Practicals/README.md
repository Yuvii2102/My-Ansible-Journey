# Ansible – AWS Collections, Ansible Vault, Variables and Jinja2 (Practicals 1–5)

## Introduction

In this episode, I practised important Ansible concepts that help make automation more secure, flexible and reusable.

I completed five practicals:

1. Installing AWS Collections
2. Verifying Boto3
3. Encrypting sensitive information using Ansible Vault
4. Working with Variables and Jinja2
5. Understanding Variable Precedence

All practicals were performed on an AWS EC2 Ubuntu Control Node.

This documentation explains how to connect from Windows PowerShell and execute the practicals on the Ubuntu EC2 instance.

---

## Environment Setup

### Architecture

```text
Windows Machine
      |
      | SSH Connection
      |
      v
AWS EC2 Control Node
Ubuntu Server
      |
      | Ansible Commands
      |
      v
Ansible Practicals
```

### Environment Details

| Component | Configuration |
|---|---|
| Local Machine | Windows |
| Terminal | PowerShell |
| Cloud Provider | AWS |
| Control Node | EC2 Ubuntu Instance |
| Ansible | Installed |
| AWS Collection | amazon.aws |
| Python SDK | Boto3 |
| Editor | Vim |
| Connection | SSH |

### Step 1: Connect to the EC2 Control Node

Open Windows PowerShell.

Use the following command to connect to the Ubuntu EC2 instance:

```powershell
ssh -i "C:\Users\Yuvraj\Desktop\ansible-key.pem" ubuntu@54.234.247.74
```

**Explanation:**

- `ssh` – Establishes an SSH connection.
- `-i` – Specifies the private key file.
- `ansible-key.pem` – EC2 private key.
- `ubuntu` – Default Ubuntu username.
- `54.234.247.74` – Control Node public IP address.

After connecting successfully, the terminal will display an Ubuntu prompt similar to:

```text
ubuntu@ip-172-31-17-219:~$
```

**Important:** The above public IP is from my practical environment. If the EC2 instance is stopped and restarted, its public IP may change. Use the current public IP shown in the AWS EC2 console.

### Step 2: Create a Working Directory

Execute the following commands inside the Ubuntu EC2 terminal:

```bash
mkdir ansible-vault-practical
cd ansible-vault-practical
```

This directory will contain the files created during the practicals.

---

# Practical 1 – Install AWS Collection

## Objective

Install and verify the AWS Ansible Collection.

### What is an Ansible Collection?

Ansible Collections are packages that contain reusable Ansible content such as modules, roles and plugins.

The AWS collection provides modules to automate AWS services such as EC2, S3, IAM and RDS.

### Step 1: Install the AWS Collection

Execute this command on the Ubuntu Control Node:

```bash
ansible-galaxy collection install amazon.aws
```

### Step 2: Verify the Installation

```bash
ansible-galaxy collection list amazon.aws
```

### Output

The collection was already installed in my environment.

```text
# /usr/lib/python3/dist-packages/ansible_collections

Collection   Version
----------   -------
amazon.aws   10.1.2
```

### Result

The `amazon.aws` collection was successfully verified.

It provides AWS-specific modules that can be used to automate AWS resources through AWS APIs.

---

# Practical 2 – Verify Boto3 Installation

## Objective

Verify whether Boto3 is installed on the Ansible Control Node.

### What is Boto3?

Boto3 is the AWS SDK for Python. It allows Python applications and tools to communicate with AWS services.

Some AWS Ansible collection modules use Boto3 behind the scenes to communicate with AWS APIs.

### Step 1: Check Boto3 Installation

Execute:

```bash
python3 -c "import boto3; print(boto3.__version__)"
```

### Output

```text
1.40.72
```

### Result

Boto3 was already installed on the Control Node.

The installed version was `1.40.72`, and Python successfully imported the library.

No additional installation was required.

**Remember:** Boto3 is a Python library, not an Ansible module.

---

# Practical 3 – Ansible Vault

## Objective

Create an encrypted credentials file using Ansible Vault and verify encryption and decryption.

### What is Ansible Vault?

Ansible Vault is a feature used to encrypt sensitive information such as:

- AWS Access Keys
- AWS Secret Access Keys
- Database passwords
- API tokens
- Other confidential information

Instead of storing sensitive values in plain text, we can protect them using Ansible Vault.

### Step 1: Create an Encrypted File

Execute:

```bash
ansible-vault create aws_credentials.yml
```

Ansible will ask you to enter a new Vault password.

```text
New Vault password:
Confirm New Vault password:
```

Enter a password and remember it.

The Vim editor will open.

### Step 2: Add Dummy Credentials

Inside Vim, press `i` to enter Insert Mode.

Add the following YAML content:

```yaml
aws_access_key: "DUMMY_ACCESS_KEY"
aws_secret_key: "DUMMY_SECRET_KEY"
```

Save and exit:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

Ansible Vault encrypts the file when it is saved.

**Note:** Dummy credentials were used for this practical. Actual AWS credentials should never be exposed in documentation or GitHub repositories.

### Step 3: Verify Encryption

Execute:

```bash
cat aws_credentials.yml
```

### Expected Output

The file begins with:

```text
$ANSIBLE_VAULT;1.1;AES256
```

The remaining content appears as encrypted hexadecimal data.

This confirms that the file is encrypted.

### Step 4: View the Decrypted Content

Execute:

```bash
ansible-vault view aws_credentials.yml
```

Enter the Vault password when prompted.

### Output

```yaml
aws_access_key: "DUMMY_ACCESS_KEY"
aws_secret_key: "DUMMY_SECRET_KEY"
```

This confirms that Ansible Vault can decrypt and display the original content.

### Important Ansible Vault Commands

| Command | Purpose |
|---|---|
| `ansible-vault create file.yml` | Create an encrypted file |
| `ansible-vault view file.yml` | View decrypted content |
| `ansible-vault edit file.yml` | Edit an encrypted file |
| `ansible-vault encrypt file.yml` | Encrypt an existing file |
| `ansible-vault decrypt file.yml` | Decrypt a file |

### Result

Successfully created an encrypted credentials file and verified both encryption and decryption using Ansible Vault.

---

# Practical 4 – Variables and Jinja2

## Objective

Create an external variables file and use Jinja2 templating to dynamically display variable values through an Ansible playbook.

### What are Variables?

Variables allow us to store values and reuse them throughout our playbooks.

They help avoid hardcoding and make playbooks reusable and easier to maintain.

### What is Jinja2?

Jinja2 is a templating engine used by Ansible to dynamically insert variable values.

The syntax is:

```jinja2
{{ variable_name }}
```

### Step 1: Create a Variables File

Inside the working directory, execute:

```bash
vim variables.yml
```

Press `i` and add:

```yaml
---
app_name: Ansible Demo
server_name: web-server
deployment_env: development
```

Save and exit:

```vim
:wq
```

### Step 2: Create the Ansible Playbook

Execute:

```bash
vim variables-playbook.yaml
```

If pasting into Vim is difficult, we can create the file directly using a terminal heredoc:

```bash
cat > variables-playbook.yaml <<'EOF'
---
- name: Demonstrate Ansible Variables and Jinja2
  hosts: localhost
  connection: local

  vars_files:
    - variables.yml

  tasks:
    - name: Display application details
      ansible.builtin.debug:
        msg:
          - "Application Name: {{ app_name }}"
          - "Server Name: {{ server_name }}"
          - "Environment: {{ deployment_env }}"
EOF
```

### Step 3: Understand the Playbook

| Configuration | Purpose |
|---|---|
| `hosts: localhost` | Runs the playbook on the Control Node |
| `connection: local` | Uses a local connection instead of SSH |
| `vars_files` | Loads variables from an external YAML file |
| `ansible.builtin.debug` | Displays information in the terminal |
| `{{ app_name }}` | Retrieves the value of a variable using Jinja2 |

**Note:** I used `deployment_env` instead of `environment` because `environment` is a reserved name in Ansible.

### Step 4: Execute the Playbook

```bash
ansible-playbook variables-playbook.yaml
```

### Expected Output

```text
PLAY [Demonstrate Ansible Variables and Jinja2]

TASK [Gathering Facts]
ok: [localhost]

TASK [Display application details]
ok: [localhost] => {
    "msg": [
        "Application Name: Ansible Demo",
        "Server Name: web-server",
        "Environment: development"
    ]
}

PLAY RECAP
localhost : ok=2 changed=0 unreachable=0 failed=0
```

### Understanding the Execution

1. Ansible reads `variables-playbook.yaml`.
2. It loads the values from `variables.yml`.
3. Jinja2 replaces the variable expressions with their actual values.
4. The debug module displays the final output.

### Result

Successfully created an external variables file and used Jinja2 templating to display dynamic values through an Ansible playbook.

---

# Practical 5 – Variable Precedence

## Objective

Understand how Ansible handles variables when the same variable is defined in different places.

### What is Variable Precedence?

Ansible allows us to define the same variable in multiple locations.

When different values are assigned to the same variable, Ansible follows precedence rules to determine which value should be used.

In this practical, I tested the difference between playbook variables and extra variables.

### Step 1: Create a Playbook

Execute:

```bash
cat > precedence-playbook.yaml <<'EOF'
---
- name: Demonstrate Variable Precedence
  hosts: localhost
  connection: local

  vars:
    instance_type: t3.small

  tasks:
    - name: Display instance type
      ansible.builtin.debug:
        msg: "Selected Instance Type: {{ instance_type }}"
EOF
```

### Step 2: Execute the Playbook

```bash
ansible-playbook precedence-playbook.yaml
```

### Output

```text
TASK [Display instance type]
ok: [localhost] => {
    "msg": "Selected Instance Type: t3.small"
}
```

The output displays `t3.small` because it is defined in the playbook's `vars` section.

### Step 3: Override the Variable Using Extra Variables

Now execute:

```bash
ansible-playbook precedence-playbook.yaml -e "instance_type=t3.medium"
```

### Output

```text
TASK [Display instance type]
ok: [localhost] => {
    "msg": "Selected Instance Type: t3.medium"
}
```

### Understanding the Result

| Variable Source | Value |
|---|---|
| Playbook variable | `t3.small` |
| Extra variable (`-e`) | `t3.medium` |
| Final selected value | `t3.medium` |

Extra variables have higher precedence than playbook variables.

Therefore, Ansible selected `t3.medium` instead of `t3.small`.

### Result

Successfully demonstrated variable precedence by overriding a playbook variable using an extra variable passed through the command line.

---

# Common Commands Used

| Command | Purpose |
|---|---|
| `mkdir` | Create a directory |
| `cd` | Change directory |
| `vim` | Open or edit a file |
| `cat` | Display file contents |
| `cat > file <<'EOF'` | Create a file using a heredoc |
| `ansible-galaxy collection install` | Install an Ansible collection |
| `ansible-galaxy collection list` | List installed collections |
| `python3 -c` | Execute a short Python command |
| `ansible-vault create` | Create an encrypted file |
| `ansible-vault view` | View encrypted file contents |
| `ansible-playbook` | Execute an Ansible playbook |
| `-e` | Pass extra variables to a playbook |

---

# Important Learnings

After completing these five practicals, I learned:

1. How to install and verify Ansible Collections.
2. How AWS collection modules communicate with AWS APIs.
3. How Boto3 helps Python applications and AWS modules interact with AWS.
4. How to encrypt sensitive information using Ansible Vault.
5. How to create and use external variable files.
6. How Jinja2 dynamically inserts variable values.
7. How variable precedence works in Ansible.
8. How extra variables override lower-precedence values.
9. How to create and execute Ansible playbooks on an Ubuntu EC2 Control Node.

---

# Final Result

Successfully completed five Ansible practicals covering AWS Collections, Boto3, Ansible Vault, Variables, Jinja2 Templating and Variable Precedence.

These exercises helped me understand how to write more secure, flexible and reusable Ansible automation.

## References

- [Ansible Collections Documentation](https://docs.ansible.com/ansible/latest/collections_guide/index.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [AWS Collection – amazon.aws](https://galaxy.ansible.com/ui/repo/published/amazon/aws/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- [Ansible Variables Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
- [Ansible Templating Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html)
- [Ansible Variable Precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

---

**Note:** All Ansible commands and practical exercises were executed on the Ubuntu EC2 Control Node. Windows PowerShell was used to establish the SSH connection to the instance. The public IP and private key path shown in this documentation belong to my practical environment and should be replaced with the current values when reproducing the exercises.
