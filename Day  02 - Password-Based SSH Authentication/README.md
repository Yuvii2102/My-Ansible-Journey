# Practical 2: Password-Based SSH Authentication

## 1. Objective

Configure password-based SSH authentication between an Ansible Control Node and two Managed Nodes.

Instead of using SSH keys, we will authenticate using the Ubuntu username and password.

After configuration, the Control Node should be able to connect to both Managed Nodes by entering their respective passwords.

## 2. Lab Architecture

We are using three Ubuntu EC2 instances.

| Instance | Role | Private IP |
|---|---|---|
| EC2 Instance 1 | Control Node | 172.31.28.28 |
| EC2 Instance 2 | Managed Node 1 | 172.31.31.231 |
| EC2 Instance 3 | Managed Node 2 | 172.31.26.245 |

**Username:** ubuntu

### Architecture

```text
             CONTROL NODE
              172.31.28.28
                   |
              SSH Connection
                   |
           Username + Password
                   |
              /         \
             /           \
            v             v
     MANAGED NODE 1   MANAGED NODE 2
      172.31.31.231    172.31.26.245
            |               |
       Password Auth   Password Auth
            |               |
            v               v
        Successful       Successful
         Login             Login
```

## 3. Prerequisites

Before starting, ensure:

- Three Ubuntu EC2 instances are running.
- SSH key-based authentication is already configured.
- Ansible is installed on the Control Node.
- SSH port 22 is allowed in the Security Group.
- You have access to both Managed Nodes.

**Important:** We will retain SSH key authentication and enable password authentication alongside it.

## 4. Step 1: Connect to Managed Node 1

First, connect to the Control Node.

From the Control Node, connect to Managed Node 1 using the SSH private key:

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.31.231
```

You are now inside Managed Node 1.

## 5. Step 2: Set a Password for the Ubuntu User

Execute:

```bash
sudo passwd ubuntu
```

You will be prompted to enter a new password.

```text
New password:
Retype new password:
```

Enter your password and confirm it.

**Note:** The password will not be visible while typing. This is normal Linux behaviour.

## 6. Step 3: Enable Password Authentication

Open the SSH server configuration file:

```bash
sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

Find the following setting:

```text
PasswordAuthentication no
```

Change it to:

```text
PasswordAuthentication yes
```

Save and exit:

```text
:wq
```

**Note:** The configuration file can differ between Ubuntu images. Check the active SSH configuration before editing:

```bash
sudo grep -RniE 'PasswordAuthentication|KbdInteractiveAuthentication' /etc/ssh/sshd_config /etc/ssh/sshd_config.d/
```

Edit the file containing the applicable setting.

## 7. Step 4: Validate SSH Configuration

Before restarting SSH, check whether the configuration has any syntax errors.

```bash
sudo sshd -t
```

If there is no output, the configuration syntax is valid.

Now verify the effective password authentication setting:

```bash
sudo sshd -T | grep passwordauthentication
```

Expected output:

```text
passwordauthentication yes
```

## 8. Step 5: Restart the SSH Service

Apply the configuration changes:

```bash
sudo systemctl restart ssh
```

Check the SSH service status:

```bash
sudo systemctl status ssh
```

Expected status:

```text
Active: active (running)
```

Managed Node 1 is now configured for password-based authentication.

## 9. Step 6: Configure Managed Node 2

Return to the Control Node:

```bash
exit
```

Connect to Managed Node 2:

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.26.245
```

Repeat the same configuration steps.

### Set the Ubuntu password

```bash
sudo passwd ubuntu
```

### Edit SSH configuration

```bash
sudo vim /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

Change:

```text
PasswordAuthentication no
```

To:

```text
PasswordAuthentication yes
```

Save and exit using:

```text
:wq
```

### Validate configuration

```bash
sudo sshd -t
```

### Verify password authentication

```bash
sudo sshd -T | grep passwordauthentication
```

Expected output:

```text
passwordauthentication yes
```

### Restart SSH

```bash
sudo systemctl restart ssh
```

Check service status:

```bash
sudo systemctl status ssh
```

Managed Node 2 is now configured for password-based authentication.

## 10. Step 7: Test Password-Based Authentication

Return to the Control Node:

```bash
exit
```

We will now test both Managed Nodes without using SSH keys.

### Test Managed Node 1

```bash
ssh -o PreferredAuthentications=password \
-o PubkeyAuthentication=no \
ubuntu@172.31.31.231
```

The terminal will prompt:

```text
ubuntu@172.31.31.231's password:
```

Enter the password configured on Managed Node 1.

### Test Managed Node 2

First, exit Managed Node 1:

```bash
exit
```

Now connect to Managed Node 2:

```bash
ssh -o PreferredAuthentications=password \
-o PubkeyAuthentication=no \
ubuntu@172.31.26.245
```

Enter the password configured on Managed Node 2.

If both connections succeed, password-based SSH authentication is working correctly.

## 11. Understanding the SSH Commands

| Command | Purpose |
|---|---|
| `sudo passwd ubuntu` | Set or change Ubuntu user password |
| `sudo vim` | Edit SSH configuration |
| `PasswordAuthentication yes` | Enable password-based SSH login |
| `sudo sshd -t` | Validate SSH configuration syntax |
| `sudo sshd -T` | Display effective SSH configuration |
| `sudo systemctl restart ssh` | Restart SSH service |
| `sudo systemctl status ssh` | Check SSH service status |
| `PreferredAuthentications=password` | Prefer password authentication |
| `PubkeyAuthentication=no` | Disable public key authentication for that test connection |

## 12. Troubleshooting

### Problem 1: Permission denied

Possible reasons:

- Incorrect username or password.
- Password authentication is disabled.
- SSH configuration was not applied.

Solution:

Verify:

```bash
sudo sshd -T | grep passwordauthentication
```

Expected:

```text
passwordauthentication yes
```

### Problem 2: SSH configuration error

If the SSH configuration contains syntax errors, validate it using:

```bash
sudo sshd -t
```

Correct the reported errors before restarting SSH.

### Problem 3: Password authentication still fails

Check:

```bash
sudo passwd -S ubuntu
```

Ensure the account has a valid password and is not locked.

Also check whether another SSH configuration setting overrides the password authentication setting.

## 13. Final Result

Successfully configured password-based SSH authentication between:

- Control Node → Managed Node 1
- Control Node → Managed Node 2

Both Managed Nodes accepted SSH connections using their Ubuntu username and password without using the SSH private key.

### Key Takeaways

1. SSH supports key-based and password-based authentication.
2. `passwd` is used to set a Linux user password.
3. `PasswordAuthentication yes` enables password-based SSH login.
4. `sshd -t` checks SSH configuration syntax.
5. `sshd -T` displays the effective SSH settings.
6. Password authentication can be tested by disabling public key authentication for an individual SSH connection.

**Practical 2 Completed: Password-Based SSH Authentication.**

### Security Note

Password authentication increases the attack surface compared with SSH key authentication. For production environments, prefer SSH keys, restrict SSH access through Security Groups, and use strong passwords if password authentication is required.
