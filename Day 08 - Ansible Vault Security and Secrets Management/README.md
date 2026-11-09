# Ansible Vault – Securing Sensitive Data

## 1. Introduction

While working with Ansible, we often deal with sensitive information such as passwords, API tokens, AWS Access Keys and Secret Keys.

Storing these values directly inside playbooks or configuration files is not a good security practice. If we upload those files to GitHub, our sensitive information could be exposed.

To solve this problem, Ansible provides a built-in feature called **Ansible Vault**.

Ansible Vault allows us to encrypt sensitive files and variables so that only someone with the correct Vault password can access them.

## 2. What is Ansible Vault?

Ansible Vault is a security feature in Ansible used to encrypt and protect sensitive information.

It can be used to protect:

- AWS Access Keys and Secret Keys
- Database passwords
- API tokens
- Application credentials
- Sensitive configuration files
- Variables used in Ansible playbooks

Ansible Vault comes built into Ansible. We do not need to install it separately.

Check the available commands:

```bash
ansible-vault --help
```

## 3. Why Do We Need Ansible Vault?

Consider a situation where we create an EC2 instance using Ansible.

To communicate with AWS, we need AWS credentials.

If we directly add the credentials inside our playbook, anyone who can access the repository may be able to see them.

For example:

```yaml
aws_access_key: "YOUR_ACCESS_KEY"
aws_secret_key: "YOUR_SECRET_KEY"
```

This creates a security risk, especially when sharing code through GitHub.

Instead, we can store these credentials in a separate YAML file and encrypt that file using Ansible Vault.

This allows us to use sensitive information in our playbooks without keeping the actual values in plain text.

## 4. How Ansible Vault Works

Ansible Vault works similarly to a bank locker.

1. We create a Vault using a password.
2. We store sensitive information inside it.
3. Ansible encrypts the information.
4. Whenever we need to access it, we provide the correct Vault password.
5. Ansible decrypts the information for use.

The encrypted file cannot be read normally without decrypting it with the correct password.

## 5. Creating an Encrypted File

The `create` command is used to create a new encrypted file.

### Command

```bash
ansible-vault create credentials.yaml
```

Ansible asks us to enter a new Vault password:

```text
New Vault password:
Confirm New Vault password:
```

After entering the password, Ansible opens the file in the configured text editor.

Add the following variables:

```yaml
ec2_access_key: "YOUR_ACCESS_KEY"
ec2_secret_key: "YOUR_SECRET_KEY"
```

Save and exit the editor.

The file is now encrypted.

## 6. Encrypting an Existing File

Sometimes we may already have a YAML file containing sensitive information.

Instead of creating a new file, we can encrypt the existing file.

### Command

```bash
ansible-vault encrypt credentials.yaml
```

Ansible asks for a Vault password and encrypts the file.

### Difference Between Create and Encrypt

| Command | Purpose |
|---|---|
| `ansible-vault create` | Creates a new encrypted file. |
| `ansible-vault encrypt` | Encrypts an existing plain-text file. |

## 7. Viewing an Encrypted File

If we want to view the contents of an encrypted file without permanently decrypting it, we can use the `view` command.

### Command

```bash
ansible-vault view credentials.yaml
```

Enter the Vault password.

Ansible displays the decrypted contents in the terminal. The original file remains encrypted.

## 8. Editing an Encrypted File

If we want to add new variables or update existing values, we can use the `edit` command.

### Command

```bash
ansible-vault edit credentials.yaml
```

After entering the Vault password, Ansible opens the file.

For example, we can add another variable:

```yaml
ec2_access_key: "YOUR_ACCESS_KEY"
ec2_secret_key: "YOUR_SECRET_KEY"
api_token: "YOUR_API_TOKEN"
```

Save and exit.

Ansible automatically encrypts the file again.

We do not need to manually encrypt it after editing.

## 9. Decrypting an Encrypted File

If we want to convert an encrypted file back into plain text, we can use the `decrypt` command.

### Command

```bash
ansible-vault decrypt credentials.yaml
```

Enter the Vault password.

The file will now contain readable information.

**Important:** After decryption, the file is no longer protected by Ansible Vault. Make sure it is not accidentally committed to GitHub.

## 10. Encrypting a Single String

Ansible Vault also allows us to encrypt an individual string instead of an entire file.

### Command

```bash
ansible-vault encrypt_string 'my-secret-token'
```

We can also provide a variable name:

```bash
ansible-vault encrypt_string --name 'api_token' 'my-secret-token'
```

Ansible generates an encrypted value that can be used inside a YAML file or playbook.

This is useful when we only want to protect a particular variable.

## 11. Using a Vault Password File

Normally, Ansible asks us to enter the Vault password whenever we perform an operation on an encrypted file.

To avoid entering it manually every time, we can store the Vault password in a separate file.

### Step 1: Create a Password File

```bash
vim vault.pass
```

Enter your Vault password and save the file.

### Step 2: Set File Permissions

```bash
chmod 600 vault.pass
```

This restricts access to the password file.

### Step 3: Use the Password File

We can now provide the password file using the `--vault-password-file` option.

Create an encrypted file:

```bash
ansible-vault create credentials.yaml --vault-password-file vault.pass
```

Encrypt an existing file:

```bash
ansible-vault encrypt credentials.yaml --vault-password-file vault.pass
```

View an encrypted file:

```bash
ansible-vault view credentials.yaml --vault-password-file vault.pass
```

Edit an encrypted file:

```bash
ansible-vault edit credentials.yaml --vault-password-file vault.pass
```

Decrypt an encrypted file:

```bash
ansible-vault decrypt credentials.yaml --vault-password-file vault.pass
```

The password file makes the process easier because Ansible reads the password directly from it.

## 12. Generating a Strong Vault Password

We should avoid using simple passwords to protect sensitive information.

OpenSSL can generate a random string that can be used as a Vault password.

### Command

```bash
openssl rand -base64 32
```

This generates random bytes and converts them into a Base64-encoded string.

We can use the generated string as our Vault password.

**Note:** Base64 is an encoding method, not encryption. The generated random string is useful as a strong password, but we must still protect it.

## 13. Using Ansible Vault in a Playbook

This is one of the most important parts of Ansible Vault.

We can load variables from an encrypted YAML file directly into an Ansible playbook.

Consider the following project structure:

```text
ansible-project/
|
|-- ec2-create.yaml
|-- credentials.yaml
|-- vault.pass
```

The `credentials.yaml` file contains our AWS credentials and is encrypted using Ansible Vault.

### Playbook

```yaml
---
- name: Create EC2 Instance
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - credentials.yaml

  tasks:
    - name: Create EC2 instance
      amazon.aws.ec2_instance:
        name: Ansible-Instance
        instance_type: t2.micro
        region: us-east-1
        image_id: ami-example
        aws_access_key: "{{ ec2_access_key }}"
        aws_secret_key: "{{ ec2_secret_key }}"
        state: present
```

### Understanding vars_files

```yaml
vars_files:
  - credentials.yaml
```

This tells Ansible to load variables from the encrypted credentials file.

The following variables retrieve the values stored in that file:

```yaml
aws_access_key: "{{ ec2_access_key }}"
aws_secret_key: "{{ ec2_secret_key }}"
```

Ansible uses the Jinja2 variable syntax to access those values.

### Execute the Playbook

```bash
ansible-playbook ec2-create.yaml --vault-password-file vault.pass
```

Ansible will:

1. Read the playbook.
2. Load the encrypted credentials file.
3. Read the Vault password.
4. Decrypt the variables for use.
5. Execute the EC2 provisioning task.

We do not need to manually decrypt the credentials file before running the playbook.

## 14. Using Ansible Vault with group_vars

Ansible also supports storing variables inside the `group_vars` directory.

Example structure:

```text
ansible-project/
|
|-- inventory
|-- playbook.yaml
|
|-- group_vars/
    |
    |-- all/
        |
        |-- credentials.yaml
```

The `group_vars/all` directory contains variables that are available to all hosts in the inventory.

We can store encrypted variable files inside this directory.

Ansible automatically loads the relevant group variables when executing a playbook.

This is useful when multiple playbooks need access to common variables.

## 15. Ansible Vault Best Practices

While working with sensitive information, we should follow these practices.

### 1. Use Strong Passwords

Avoid simple passwords such as `admin123` or `password123`.

Use strong and randomly generated passwords.

### 2. Protect the Vault Password

The Vault password is just as important as the encrypted file.

If someone gets access to both the encrypted file and its password, they can decrypt the information.

Never upload the Vault password file to GitHub.

### 3. Use Secret Management Services

For production environments, we should consider using dedicated secret management services.

Examples:

- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- Azure Key Vault

These services help securely store secrets and control who can access them.

### 4. Separate Environment Credentials

Organizations usually have different environments such as Development, Staging and Production.

We can use separate Vault passwords for each environment.

| Environment | Vault Password |
|---|---|
| Development | Development Vault password |
| Staging | Staging Vault password |
| Production | Production Vault password |

This helps limit access to sensitive production information.

### 5. Never Commit Sensitive Files

Add sensitive files to `.gitignore`:

```gitignore
vault.pass
password.txt
password.yaml
credentials.yaml
*.pem
```

Remember that `.gitignore` does not remove files that were already tracked by Git. If a secret was committed, it must be handled separately.

## 16. Important Ansible Vault Commands

| Command | Purpose |
|---|---|
| `ansible-vault create file.yaml` | Create a new encrypted file |
| `ansible-vault encrypt file.yaml` | Encrypt an existing file |
| `ansible-vault decrypt file.yaml` | Decrypt an encrypted file |
| `ansible-vault edit file.yaml` | Edit an encrypted file |
| `ansible-vault view file.yaml` | View decrypted contents |
| `ansible-vault encrypt_string` | Encrypt an individual string |
| `ansible-vault --help` | Display available commands |

## 17. Practical Connection to My Previous Project

I already used Ansible Vault while working on my Ansible Real-Time AWS EC2 Provisioning and Management Project.

In that project, I created two files:

- `password.yaml` – Stored AWS Access Key and Secret Key.
- `password.txt` – Stored the Ansible Vault password.

I encrypted the credentials file using:

```bash
ansible-vault encrypt password.yaml --vault-password-file password.txt
```

Inside my EC2 provisioning playbook, I loaded the encrypted file:

```yaml
vars_files:
  - password.yaml
```

I accessed the credentials using:

```yaml
aws_access_key: "{{ ec2_access_key }}"
aws_secret_key: "{{ ec2_secret_key }}"
```

Finally, I executed the playbook using:

```bash
ansible-playbook ec2-instance-create.yaml --vault-password-file password.txt
```

This allowed me to use AWS credentials without storing them as plain text inside the playbook.

## 18. Key Learnings

Through this topic, I learned:

- What Ansible Vault is and why it is used.
- How to encrypt and decrypt sensitive files.
- How to create, view and edit encrypted files.
- How to encrypt individual strings.
- How to generate a strong Vault password.
- How to use a Vault password file.
- How to load encrypted variables into Ansible playbooks.
- How to use Vault with `group_vars`.
- Security best practices for managing sensitive information.

## 19. Conclusion

Ansible Vault is an important feature for securing sensitive information in Ansible projects.

Instead of storing passwords, API tokens and AWS credentials directly in playbooks, we can encrypt them and securely use them whenever required.

As a DevOps engineer, managing secrets properly is an important part of building secure automation workflows.

**Reference Project:**  
[My Ansible Real-Time AWS EC2 Provisioning and Management Project](https://github.com/Yuvii2102/My-Complete-DevOps-Projects)
