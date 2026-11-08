# Practical 1: Passwordless Authentication Using SSH Keys

## 1. Objective

Configure passwordless SSH authentication between an Ansible Control Node and two Managed Nodes using SSH key-based authentication.

After configuration, the Control Node should be able to connect to both Managed Nodes without entering an SSH login password or specifying the original AWS PEM key for every connection.

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
             Generate SSH Key
                   |
             Private + Public Key
                   |
            Copy Public Key
              /         \
             /           \
            v             v
     MANAGED NODE 1   MANAGED NODE 2
      172.31.31.231    172.31.26.245
            |               |
      authorized_keys  authorized_keys
            |               |
            v               v
       Passwordless     Passwordless
        SSH Login        SSH Login
```

## 3. Prerequisites

Before starting, ensure:

- Three Ubuntu EC2 instances are running.
- Ansible is installed on the Control Node.
- All three instances can communicate over the network.
- SSH (port 22) is allowed in the Security Group.
- You have the original AWS PEM key.
- The PEM key is available on the Control Node.

**Important:** The PEM key is used for the initial SSH connection. We will generate a separate SSH key pair for passwordless authentication.

## 4. Step 1: Connect to the Control Node

Connect to your Control Node using AWS EC2 Instance Connect or SSH.

Verify the instance:

```bash
hostname -I
```

Expected private IP:

```text
172.31.28.28
```

Update Ubuntu packages:

```bash
sudo apt update
```

## 5. Step 2: Generate an SSH Key Pair

SSH key-based authentication uses two keys:

- Private Key: Stays on the Control Node.
- Public Key: Copied to the Managed Nodes.

Generate an SSH key pair:

```bash
ssh-keygen -t ed25519
```

When prompted for the file location, press Enter to use the default location.

```text
/home/ubuntu/.ssh/id_ed25519
```

When asked for a passphrase, press Enter twice to leave it empty for this lab.

### Verify the generated keys

```bash
ls -l ~/.ssh/
```

You should see:

```text
id_ed25519
id_ed25519.pub
```

### Understand the files

| File | Description |
|---|---|
| id_ed25519 | Private key |
| id_ed25519.pub | Public key |
| authorized_keys | Stores trusted public keys on the Managed Node |
| known_hosts | Stores fingerprints of previously contacted SSH servers |

**Important:** Never share or copy your private key to the Managed Nodes.

## 6. Step 3: Copy the Public Key to Managed Node 1

We use `ssh-copy-id` to copy the public key to the Managed Node.

Command:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub \
-o IdentityFile=/home/ubuntu/new.pem \
-o IdentitiesOnly=yes \
ubuntu@172.31.31.231
```

Explanation:

- `ssh-copy-id`: Copies the public key to a remote machine.
- `-i`: Specifies the public key file.
- `IdentityFile`: Specifies the original AWS PEM key used for initial authentication.
- `IdentitiesOnly=yes`: Uses the specified identity file.
- `ubuntu`: Remote username.
- `172.31.31.231`: Managed Node 1 private IP.

If prompted to confirm the host fingerprint, type:

```text
yes
```

The public key will be added to:

```text
/home/ubuntu/.ssh/authorized_keys
```

## 7. Step 4: Copy the Public Key to Managed Node 2

Repeat the same process for Managed Node 2.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub \
-o IdentityFile=/home/ubuntu/new.pem \
-o IdentitiesOnly=yes \
ubuntu@172.31.26.245
```

Now the same public key is installed on both Managed Nodes.

**Note:** Replace `/home/ubuntu/new.pem` with the actual PEM key path on your Control Node if your key has a different location.

If `ssh-copy-id` reports that the key already exists, it means the public key is already present on that remote machine.

## 8. Step 5: Verify SSH Directory Permissions

SSH requires appropriate permissions for authentication.

Connect to Managed Node 1 using the original PEM key:

```bash
ssh -i /home/ubuntu/new.pem ubuntu@172.31.31.231
```

Execute:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Exit:

```bash
exit
```

Repeat the same process for Managed Node 2:

```bash
ssh -i /home/ubuntu/new.pem ubuntu@172.31.26.245
```

Execute:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Exit:

```bash
exit
```

You should now be back on the Control Node.

## 9. Step 6: Test Passwordless SSH Authentication

Now test the SSH connection from the Control Node using the newly generated private key.

### Test Managed Node 1

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.31.231
```

### Test Managed Node 2

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.26.245
```

If both connections work without asking for an SSH login password or the original PEM key, passwordless authentication is configured successfully.

To return to the Control Node:

```bash
exit
```

## 10. Step 7: Verify the Authentication

You can also verify that SSH is using your new key with verbose output:

```bash
ssh -v -i ~/.ssh/id_ed25519 ubuntu@172.31.31.231
```

Look for authentication success messages in the output.

## 11. Important SSH Commands

| Command | Purpose |
|---|---|
| `ssh-keygen -t ed25519` | Generate SSH key pair |
| `ls -l ~/.ssh/` | List SSH files |
| `cat ~/.ssh/id_ed25519.pub` | Display public key |
| `ssh-copy-id` | Copy public key to remote machine |
| `ssh -i` | Connect using a specific private key |
| `chmod 700 ~/.ssh` | Set SSH directory permissions |
| `chmod 600 ~/.ssh/authorized_keys` | Set authorized_keys permissions |
| `exit` | Disconnect from remote machine |

## 12. Troubleshooting

### Problem 1: Permission denied (publickey)

Possible reasons:

- Public key is not installed correctly.
- Incorrect SSH username.
- Incorrect private key.
- Wrong permissions on `.ssh` or `authorized_keys`.

Solution:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Check that the public key generated on the Control Node exists in the Managed Node's `authorized_keys` file.

### Problem 2: Identity file not accessible

Error:

```text
Identity file not accessible: No such file or directory
```

Reason:

The private key is missing from the machine where the SSH command is being executed.

Solution:

Run the SSH command from the Control Node, where the private key is stored.

### Problem 3: Connection timed out

Possible reasons:

- Security Group does not allow SSH.
- Incorrect private IP.
- EC2 instance is stopped.
- Network connectivity issue.

Solution:

Check the Security Group inbound rules and ensure TCP port 22 is allowed from the Control Node.

## 13. Final Result

Successfully configured SSH key-based passwordless authentication between:

- Control Node → Managed Node 1
- Control Node → Managed Node 2

Both Managed Nodes now trust the public key generated by the Control Node.

### Key Takeaways

1. SSH key-based authentication uses a public key and a private key.
2. The private key stays on the Control Node.
3. The public key is copied to the Managed Nodes.
4. `ssh-copy-id` simplifies public key distribution.
5. Passwordless SSH authentication is useful for Ansible because Ansible needs to connect to Managed Nodes to execute tasks remotely.

**Practical 1 Completed: SSH Key-Based Passwordless Authentication.**
