# Ansible – AWS Collections, Ansible Vault, Variables and Jinja2

## Introduction

In this episode, I learned how Ansible can be used to automate AWS resources and how to make playbooks more secure, flexible and reusable.

The main concepts covered are:

1. Ansible Collections
2. Built-in Modules vs AWS Collection Modules
3. Boto3
4. Ansible Vault
5. Variables in Ansible
6. Jinja2 Templating
7. Variable Precedence

---

## 1. Ansible Collections

### What is a Collection?

Ansible Collections are packages that contain reusable Ansible content, such as:

- Modules
- Roles
- Plugins
- Playbooks

Think of a collection as a toolkit that provides additional capabilities to Ansible.

For example, Ansible has built-in modules to manage Linux servers. But if we want to create an EC2 instance or manage an S3 bucket, we can use the AWS collection.

### Important AWS Collection

`amazon.aws`

This collection provides modules to automate AWS services such as:

- EC2
- S3
- IAM
- RDS
- Route 53

### Installing the AWS Collection

```bash
ansible-galaxy collection install amazon.aws
```

**Remember:** Collections extend Ansible's capabilities by providing additional modules and other reusable content.

---

## 2. Built-in Modules vs AWS Collection Modules

Ansible can communicate with different systems depending on what we want to automate.

### A. Built-in Modules

Built-in modules are commonly used to manage the configuration of remote servers.

Examples:

| Module | Purpose |
|---|---|
| `apt` | Install packages |
| `copy` | Copy files |
| `file` | Manage files and directories |
| `service` | Manage services |

Example:

```yaml
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
```

These modules typically execute tasks on managed nodes through SSH.

### B. AWS Collection Modules

AWS collection modules are used to manage AWS resources through AWS APIs.

Examples:

- Create an EC2 instance.
- Create an S3 bucket.
- Manage IAM resources.

Example module:

```yaml
amazon.aws.ec2_instance
```

### How They Communicate

**Built-in Modules – Server Configuration**

```text
Ansible Control Node
        |
        | SSH Connection
        |
        v
   Managed Node
        |
        v
Install Apache, manage files,
packages and services
```

**AWS Collection – Cloud Resource Management**

```text
Ansible Control Node
        |
        | AWS API
        |
        v
       AWS
        |
        v
Manage EC2, S3, IAM
and other AWS resources
```

**Interview Point:** Built-in modules commonly configure remote machines through SSH, whereas AWS collection modules communicate with AWS services through their APIs.

---

## 3. Boto3

### What is Boto3?

Boto3 is the AWS SDK (Software Development Kit) for Python.

It allows Python applications and tools to communicate with AWS services.

Using Boto3, we can:

- Create EC2 instances.
- Upload files to S3.
- Manage AWS resources.
- Retrieve information from AWS.

### Why Do We Need Boto3 in Ansible?

Some AWS collection modules use Boto3 behind the scenes to communicate with AWS APIs.

Think of it this way:

- **Ansible:** Provides the automation instructions.
- **AWS Collection:** Provides AWS-specific modules.
- **Boto3:** Helps those modules communicate with AWS.

**Important:** Boto3 is a Python library, not an Ansible module.

---

## 4. Ansible Vault

### What is Ansible Vault?

Ansible Vault is a feature used to encrypt sensitive information.

Examples of sensitive information include:

- AWS Access Keys
- AWS Secret Access Keys
- Database passwords
- API tokens
- Other confidential information

Imagine we are writing an Ansible playbook to create an EC2 instance.

AWS requires authentication. We should never directly write our secret credentials inside the playbook.

Instead, we can store them in an encrypted file using Ansible Vault.

### How Does Ansible Vault Work?

```text
Sensitive Credentials
(Access Key and Secret Key)
          |
          v
     Ansible Vault
          |
          v
 Encrypts the credentials file
          |
          v
  Encrypted Credentials File
```

### Important Ansible Vault Commands

**1. Create an encrypted file**

```bash
ansible-vault create aws_credentials.yml
```

**2. View an encrypted file**

```bash
ansible-vault view aws_credentials.yml
```

**3. Edit an encrypted file**

```bash
ansible-vault edit aws_credentials.yml
```

**4. Encrypt an existing file**

```bash
ansible-vault encrypt aws_credentials.yml
```

**5. Decrypt a file**

```bash
ansible-vault decrypt aws_credentials.yml
```

### Security Points

- Never expose AWS credentials in playbooks.
- Never commit your Vault password file to GitHub.
- Base64 encoding is not encryption.
- Ansible Vault performs the actual encryption.

**Remember:** Ansible Vault helps protect sensitive information used in automation.

---

## 5. Variables in Ansible

### What are Variables?

Variables allow us to store values and reuse them throughout our playbooks.

Imagine we have a playbook that creates an EC2 instance.

Instead of hardcoding the instance type everywhere, we can define it as a variable.

Example:

```yaml
instance_type: t3.small
```

Now we can change the instance type in one place without rewriting the entire playbook.

### Why Are Variables Important?

- Avoid hardcoding values.
- Make playbooks reusable.
- Make configuration easier to maintain.
- Allow different environments to use different values.

### Important Places to Define Variables

| Location | Purpose |
|---|---|
| `defaults/main.yml` | Defines default values for a role |
| `vars/main.yml` | Defines role-specific variables |
| `group_vars/` | Defines variables for groups of hosts |
| Playbook `vars:` | Defines variables inside a playbook |
| Extra variables `-e` | Passes values while running a playbook |

**Remember:** Variables make Ansible playbooks flexible and reusable.

---

## 6. Jinja2 Templating

### What is Jinja2?

Jinja2 is a templating engine used by Ansible to dynamically insert variable values into playbooks and templates.

In simple words, it allows Ansible to use variables dynamically.

### Important Jinja2 Syntax

```jinja2
{{ variable_name }}
```

### Example 1: Using a Variable

```yaml
instance_type: "{{ ec2_instance_type }}"
```

Here, Ansible replaces `{{ ec2_instance_type }}` with the value stored in that variable.

### Example 2: Another Variable

```yaml
name: "{{ server_name }}"
```

If the variable is:

```yaml
server_name: web-server
```

The final value becomes:

```yaml
name: web-server
```

**Remember:** Jinja2 helps make Ansible playbooks dynamic and reusable.

---

## 7. Variable Precedence

### What is Variable Precedence?

Ansible allows us to define the same variable in different locations.

But what happens if the same variable has different values?

Ansible follows variable precedence rules to decide which value should be used.

### Example

Imagine we define the same variable in two different files.

**In `defaults/main.yml`:**

```yaml
instance_type: t2.micro
```

**In `vars/main.yml`:**

```yaml
instance_type: t3.small
```

When both are applicable, the role variable has higher precedence than the role default.

So Ansible uses:

```yaml
instance_type: t3.small
```

### Important Variable Precedence Levels

| Variable Location | General Priority |
|---|---|
| Role defaults | Low |
| Role vars | Higher than role defaults |
| Playbook vars | Higher than role defaults |
| Extra variables (`-e`) | Very high |

This is a simplified view, not the complete Ansible precedence list.

### Example: Passing Extra Variables

We can pass a variable directly while running a playbook.

```bash
ansible-playbook playbook.yml -e "instance_type=t3.medium"
```

Extra variables have very high precedence and override lower-precedence values.

**Remember:** We don't need to memorize all 22 precedence categories. We should understand the concept and know that extra variables have very high precedence.

---

## Key Takeaways

After learning these concepts, I understood how Ansible can be used beyond configuring Linux servers.

| Concept | What I Learned |
|---|---|
| Collections | Extend Ansible with reusable content |
| AWS Collection | Automate AWS resources |
| Boto3 | Python SDK for communicating with AWS |
| Ansible Vault | Encrypt sensitive information |
| Variables | Store and reuse values |
| Jinja2 | Dynamically insert variable values |
| Variable Precedence | Decide which variable value Ansible uses |

## Important Topics to Focus On

**High Priority**

- Ansible Collections
- Built-in modules vs AWS collection modules
- Boto3 basics
- Ansible Vault
- Variables
- Jinja2 templating
- Variable precedence

**Medium Priority**

- `defaults/main.yml`
- `vars/main.yml`
- `group_vars/`
- Playbook variables
- Extra variables
- `localhost` and `connection: local`

**Learn Later**

- All variable precedence categories
- Every AWS collection module
- Advanced Jinja2 features
- Every available Ansible collection

## Conclusion

This episode helped me understand how Ansible can automate AWS resources and how to write secure and reusable playbooks.

I also learned how Ansible Vault protects sensitive information and how variables, Jinja2 and variable precedence make automation more flexible.

The next step is to apply these concepts through practical exercises by automating AWS resources using Ansible.

## References

- [Ansible Collections Documentation](https://docs.ansible.com/ansible/latest/collections_guide/index.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [AWS Collection – amazon.aws](https://galaxy.ansible.com/ui/repo/published/amazon/aws/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/vault_guide/index.html)
- [Ansible Variables Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
- [Ansible Templating Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html)
- [Ansible Variable Precedence Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)
