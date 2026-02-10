
# SSH Key Generation and Configuration Guide

This guide covers the essential steps for generating SSH keys, configuring them for services like GitHub, and connecting to remote servers securely.

## Generating SSH Keys

### ED25519 Key (Recommended)

ED25519 is the modern, secure, and efficient SSH key algorithm. Use this for new key generation:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### RSA Key (Alternative)

If you need compatibility with older systems:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

When prompted:

* Accept the default location (`~/.ssh/id_ed25519`) or specify a custom path
* Enter a strong passphrase or leave empty for no passphrase (not recommended for production)

## Using SSH Keys with GitHub

### Step 1: Copy Your Public Key

Display and copy your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

### Step 2: Add to GitHub

1. Go to GitHub Settings → SSH and GPG keys
2. Click "New SSH key"
3. Give it a descriptive title (e.g., "Work Laptop")
4. Paste your public key
5. Click "Add SSH key"

### Step 3: Test the Connection

```bash
ssh -T git@github.com
```

You should see a success message confirming authentication.

## Connecting to Remote Servers

### Step 1: Copy Your Public Key to the Server

Use `ssh-copy-id` for automatic setup:

```bash
ssh-copy-id user@server_ip
```

Or manually copy the key:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

This command creates the `.ssh` directory if it doesn't exist and appends your public key to the authorized keys file.

### Step 2: Set Correct Permissions

On the remote server, ensure proper permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Step 3: Connect to the Server

```bash
ssh user@server_ip
```

## SSH Agent for Passphrase Management

If you set a passphrase on your SSH key, you can use SSH agent to avoid entering it repeatedly.

### Start SSH Agent

```bash
eval "$(ssh-agent -s)"
```

### Add Your Key to the Agent

```bash
ssh-add ~/.ssh/id_ed25519
```

### Automatic Agent Startup

To automatically start the SSH agent when you open a terminal, add the following to your `~/.bashrc` or `~/.zshrc`:

```bash
if [ -z "$SSH_AUTH_SOCK" ] ; then
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/id_ed25519
fi
```

This checks if the SSH agent is running and starts it if necessary, then adds your key automatically.

## SSH Config File

For easier server management, create an SSH config file at `~/.ssh/config`:

```
Host myserver
    HostName 192.168.1.100
    User username
    IdentityFile ~/.ssh/id_ed25519
    Port 22

Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

Now you can connect using:

```bash
ssh myserver
```

## Security Best Practices

* Always use a passphrase for SSH keys in production environments
* Never share your private key (`id_ed25519`)
* Only share your public key (`id_ed25519.pub`)
* Use different SSH keys for different purposes (work, personal, servers)
* Regularly rotate SSH keys (every 6-12 months)
* Disable password authentication on servers once SSH keys are configured
* Keep private keys secure with proper file permissions (600)

## Troubleshooting

### Permission Denied

Check file permissions:

```bash
ls -la ~/.ssh
```

Ensure:

* `~/.ssh` is 700
* `~/.ssh/authorized_keys` is 600
* `~/.ssh/id_ed25519` is 600
* `~/.ssh/id_ed25519.pub` is 644

### Agent Not Running

Restart the SSH agent:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Key Not Being Used

Specify the key explicitly:

```bash
ssh -i ~/.ssh/id_ed25519 user@server_ip
```

## Quick Reference

| Task                   | Command                                          |
| ---------------------- | ------------------------------------------------ |
| Generate ED25519 key   | `ssh-keygen -t ed25519 -C "email@example.com"` |
| View public key        | `cat ~/.ssh/id_ed25519.pub`                    |
| Copy key to server     | `ssh-copy-id user@server_ip`                   |
| Start SSH agent        | `eval "$(ssh-agent -s)"`                       |
| Add key to agent       | `ssh-add ~/.ssh/id_ed25519`                    |
| Test GitHub connection | `ssh -T git@github.com`                        |
| Connect to server      | `ssh user@server_ip`                           |
