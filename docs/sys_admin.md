# Linux System Administration Command Reference

A comprehensive collection of frequently used commands for Linux system administration, organized by category for quick reference and future expansion.

## Table of Contents

- [Network Tunneling](#network-tunneling)
- [DNS Configuration](#dns-configuration)
- [Host Management](#host-management)
- [Package Manager Configuration](#package-manager-configuration)
- [Security and Monitoring](#security-and-monitoring)
- [Proxy Configuration](#proxy-configuration)

---

## Network Tunneling

### SSH Dynamic Port Forwarding (SOCKS Proxy)

Create an SSH tunnel with dynamic port forwarding to establish a SOCKS proxy:

```bash
ssh -p 2233 -D 1080 -C -q -N rust@85.192.28.246
```

**Parameters breakdown:**
- `-p 2233` - Connect to SSH server on port 2233
- `-D 1080` - Create SOCKS proxy on local port 1080
- `-C` - Enable compression for faster data transfer
- `-q` - Quiet mode, suppress most warning messages
- `-N` - Do not execute remote commands, useful for port forwarding only

**Use case:** This creates a local SOCKS5 proxy that routes traffic through the remote server, useful for bypassing restrictions or securing connections.

---

## DNS Configuration

### Modifying System DNS Resolvers

Edit the DNS resolver configuration file:

```bash
vi /etc/resolv.conf
```

**Common configurations:**

```bash
# Primary and secondary DNS servers
nameserver 8.8.8.8
nameserver 8.8.4.4

# Alternative DNS providers
nameserver 1.1.1.1
nameserver 1.0.0.1
```

**Note:** Changes to `/etc/resolv.conf` may be temporary on systems using NetworkManager or systemd-resolved. For persistent changes, configure the appropriate network management service.

---

## Host Management

### Editing Host File Entries

Modify local hostname to IP address mappings:

```bash
vi /etc/hosts
```

**Example entries:**

```bash
# Local machine
127.0.0.1   localhost
127.0.1.1   hostname

# Custom mappings
192.168.1.100   server1.local
192.168.1.101   server2.local
10.0.0.50       database.internal
```

**Use case:** Override DNS resolution for specific domains, create shortcuts for internal servers, or block unwanted domains by mapping them to 127.0.0.1.

---

## Package Manager Configuration

### YUM Proxy Configuration (Oracle Linux 7.9 / RHEL / CentOS)

Configure YUM package manager to use a proxy server:

```bash
sudo vi /etc/yum.conf
```

**Add or modify the following configuration:**

```ini
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
proxy=socks5://127.0.0.1:1080
```

**Configuration details:**
- `gpgcheck=1` - Verify package signatures for security
- `installonly_limit=3` - Keep last 3 kernel versions
- `proxy=socks5://127.0.0.1:1080` - Route YUM traffic through SOCKS5 proxy

**Alternative proxy formats:**
```ini
proxy=http://proxy.server.com:8080
proxy=socks5://username:password@proxy.server.com:1080
```

---

## Security and Monitoring

### Fail2Ban Status Monitoring

Check Fail2Ban service status for SSH protection:

```bash
sudo fail2ban-client status sshd
```

**Output includes:**
- Currently banned IP addresses
- Total number of bans
- Currently failed login attempts

### View Fail2Ban IPTables Rules

Display firewall rules created by Fail2Ban for SSH:

```bash
sudo iptables -L f2b-sshd -n --line-numbers
```

**Parameters:**
- `-L` - List rules in the specified chain
- `-n` - Display IP addresses and ports numerically
- `--line-numbers` - Show rule numbers for easier management

### Login History

View recent user login history:

```bash
last
```

**Advanced usage:**
```bash
# Show last 20 logins
last -n 20

# Show logins for specific user
last username

# Show failed login attempts
lastb
```

---

## Proxy Configuration

### Terminal Session Proxy Environment Variables

Configure proxy settings for the current terminal session:

```bash
export http_proxy="socks5://user:pass@IP:8080"
export https_proxy="socks5://user:pass@IP:8080"
export HTTP_PROXY="socks5://user:pass@IP:8080"
export HTTPS_PROXY="socks5://user:pass@IP:8080"
```

**Supported protocols:**
```bash
# HTTP proxy
export http_proxy="http://proxy.server.com:8080"

# SOCKS5 proxy with authentication
export http_proxy="socks5://username:password@proxy.server.com:1080"

# SOCKS5 proxy without authentication
export http_proxy="socks5://127.0.0.1:1080"
```

### Making Proxy Settings Persistent

Add proxy configuration to shell profile for persistence across sessions:

```bash
# For Bash
echo 'export http_proxy="socks5://127.0.0.1:1080"' >> ~/.bashrc
echo 'export https_proxy="socks5://127.0.0.1:1080"' >> ~/.bashrc
source ~/.bashrc

# For Zsh
echo 'export http_proxy="socks5://127.0.0.1:1080"' >> ~/.zshrc
echo 'export https_proxy="socks5://127.0.0.1:1080"' >> ~/.zshrc
source ~/.zshrc
```

### Unsetting Proxy Variables

Remove proxy configuration from current session:

```bash
unset http_proxy
unset https_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
```

### No Proxy Configuration

Exclude specific domains or IP ranges from proxy:

```bash
export no_proxy="localhost,127.0.0.1,192.168.0.0/16,.internal.domain"
export NO_PROXY="localhost,127.0.0.1,192.168.0.0/16,.internal.domain"
```

---

## Additional Tips

### Testing Proxy Configuration

Verify proxy settings are working correctly:

```bash
# Test HTTP connection through proxy
curl -x socks5://127.0.0.1:1080 http://ifconfig.me

# Test HTTPS connection
curl -x socks5://127.0.0.1:1080 https://ifconfig.me

# Verbose output for debugging
curl -v -x socks5://127.0.0.1:1080 https://google.com
```

### Proxy Authentication

When using authenticated proxies, URL encode special characters in passwords:

```bash
# Space becomes %20
# @ becomes %40
# : becomes %3A

export http_proxy="socks5://user:p%40ssw0rd@proxy.server.com:1080"
```

---

