# Fail2Ban Installation and Configuration Guide

## Overview

This guide provides step-by-step instructions for installing and configuring Fail2Ban on Oracle Linux 7.9. The configuration implements a security policy that blocks IP addresses for 3 days after 5 failed authentication attempts.

## Installation Steps

### 1. Enable EPEL Repository

```bash
sudo yum install -y epel-release
```

### 2. Install Fail2Ban

```bash
sudo yum install -y fail2ban fail2ban-systemd
```

### 3. Verify Installation

```bash
fail2ban-client --version
```

## Configuration

### 1. Create Local Configuration File

Fail2Ban uses `/etc/fail2ban/jail.conf` as the default configuration. To preserve your custom settings during updates, create a local configuration file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### 2. Configure SSH Protection

Edit the local configuration file:

```bash
sudo vi /etc/fail2ban/jail.local
```

Locate the `[DEFAULT]` section and modify the following parameters:

```ini
[DEFAULT]
# Ban duration: 3 days (259200 seconds)
bantime = 4320m

# Time window to count failures: 10 minutes
findtime = 10m

# Maximum number of failures before ban
maxretry = 5

# Ban action (iptables is default)
banaction = iptables-multiport
```

### 3. Enable SSH Jail

In the same file, locate or create the `[sshd]` section:

```ini
[sshd]
enabled = true
port = ssh
maxretry = 5
logpath = %(sshd_log)s
backend = %(sshd_backend)s
bantime = 259200
findtime = 600
```

If you use a custom SSH port, replace `ssh` with your port number (e.g., `port = 2222`).

### 4. Configure Email Notifications (Optional)

To receive email notifications when IPs are banned:

```ini
[DEFAULT]
# Email configuration
destemail = your-email@example.com
sendername = Fail2Ban
mta = sendmail

# Action with email notification
action = %(action_mwl)s
```

Note: Ensure that `sendmail` or another MTA is installed and configured on your system.

## Service Management

### 1. Enable Fail2Ban Service

```bash
sudo systemctl enable fail2ban
```

### 2. Start Fail2Ban Service

```bash
sudo systemctl start fail2ban
```

### 3. Check Service Status

```bash
sudo systemctl status fail2ban
```

### 4. Restart Service After Configuration Changes

```bash
sudo systemctl restart fail2ban
```

## Verification and Monitoring

### Check Active Jails

```bash
sudo fail2ban-client status
```

### Check SSH Jail Status

```bash
sudo fail2ban-client status sshd
```

### View Banned IP Addresses

```bash
sudo fail2ban-client status sshd
```

The output will show currently banned IPs and statistics.

### Check Fail2Ban Logs

```bash
sudo tail -f /var/log/fail2ban.log
```

## Management Commands

### Manually Ban an IP Address

```bash
sudo fail2ban-client set sshd banip 192.168.1.100
```

### Manually Unban an IP Address

```bash
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

### Unban All IP Addresses

```bash
sudo fail2ban-client unban --all
```

### Reload Configuration

```bash
sudo fail2ban-client reload
```

## Firewall Configuration

Ensure that your firewall allows SSH connections and doesn't conflict with Fail2Ban:

```bash
# Check firewall status
sudo firewall-cmd --state

# Allow SSH through firewall (if needed)
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

## Whitelist Trusted IPs (Optional)

To prevent specific IP addresses from being banned, add them to the `ignoreip` parameter in `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24 YOUR_TRUSTED_IP
```

Multiple IPs can be separated by spaces.

## Testing the Configuration

### 1. Test Configuration Syntax

```bash
sudo fail2ban-client -t
```

This command validates your configuration files without starting the service.

### 2. Simulate Failed Login Attempts

From another machine, attempt to SSH with incorrect credentials 5 times to trigger a ban. Then verify the ban:

```bash
sudo fail2ban-client status sshd
```

## Troubleshooting

### Service Fails to Start

Check the logs for errors:

```bash
sudo journalctl -u fail2ban -n 50
sudo tail -50 /var/log/fail2ban.log
```

### Configuration Issues

Verify configuration syntax:

```bash
sudo fail2ban-client -t
```

### SELinux Issues

If SELinux is enforcing and blocking Fail2Ban:

```bash
sudo setsebool -P authlogin_nsswitch_use_ldap 1
sudo ausearch -c 'fail2ban-server' --raw | audit2allow -M fail2ban-local
sudo semodule -i fail2ban-local.pp
```

### Fail2Ban Not Blocking IPs

Verify that iptables is working:

```bash
sudo iptables -L -n | grep fail2ban
```

Check that the log path is correct:

```bash
ls -l /var/log/secure
```

## Advanced Configuration

### Custom Jail for Other Services

Create custom jails by adding sections to `/etc/fail2ban/jail.local`:

```ini
[custom-service]
enabled = true
port = 8080
logpath = /var/log/custom-service.log
maxretry = 5
bantime = 259200
findtime = 600
```

### Persistent Bans Across Reboots

By default, bans are cleared on service restart. To persist bans, configure a database backend in `/etc/fail2ban/fail2ban.local`:

```ini
[Definition]
dbfile = /var/lib/fail2ban/fail2ban.sqlite3
dbpurgeage = 259200
```

## Maintenance

### Regular Log Review

Periodically review Fail2Ban logs to identify attack patterns:

```bash
sudo grep "Ban" /var/log/fail2ban.log | tail -20
```

### Update Fail2Ban

Keep Fail2Ban updated for security patches:

```bash
sudo yum update fail2ban
```

### Backup Configuration

Regularly backup your configuration files:

```bash
sudo cp -r /etc/fail2ban /etc/fail2ban.backup.$(date +%Y%m%d)
```

## Security Considerations

- The 3-day ban period is aggressive; adjust based on your security requirements
- Monitor logs regularly to ensure legitimate users aren't being blocked
- Maintain a whitelist of trusted IP addresses
- Consider implementing additional security measures (key-based authentication, SSH port changes)
- Keep Fail2Ban and system packages updated

## Uninstallation (If Needed)

To remove Fail2Ban:

```bash
sudo systemctl stop fail2ban
sudo systemctl disable fail2ban
sudo yum remove fail2ban fail2ban-systemd
```

## Additional Resources

- Official Documentation: https://fail2ban.readthedocs.io
- GitHub Repository: https://github.com/fail2ban/fail2ban
- Man Pages: `man fail2ban-client`, `man jail.conf`

## License

This guide is provided as-is for educational and operational purposes.

## Contributing

Feel free to submit issues or improvements to this guide.
