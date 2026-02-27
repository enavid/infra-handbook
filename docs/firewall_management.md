# Firewall Port Management - firewalld and ufw

---

## firewalld (RHEL, Fedora, AlmaLinux, Rocky)

### Add a port

```bash
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

### Remove a port

```bash
sudo firewall-cmd --permanent --remove-port=443/tcp
sudo firewall-cmd --reload
```

### List open ports

```bash
sudo firewall-cmd --list-ports
```

### Check firewalld status

```bash
sudo systemctl status firewalld
```

### Adding ports while firewalld is stopped

Yes, this is possible. The `--permanent` flag writes the rule to disk without applying it to the running firewall. You can run permanent commands even when the service is inactive.

```bash
sudo firewall-cmd --permanent --add-port=8451/tcp
# No --reload needed. The rule will be active next time firewalld starts.
```

---

## ufw (Ubuntu, Debian)

### Add a port

```bash
sudo ufw allow 443/tcp
```

### Remove a port

```bash
sudo ufw delete allow 443/tcp
```

### List open ports

```bash
sudo ufw status
```

For a numbered list (useful when deleting rules):

```bash
sudo ufw status numbered
```

### Check ufw status

```bash
sudo ufw status verbose
```

### Adding ports while ufw is inactive

Yes, this is also possible with ufw. Rules added while ufw is inactive are stored and will apply when ufw is enabled.

```bash
sudo ufw allow 8451/tcp
# Rule is saved. It becomes active when ufw is enabled:
sudo ufw enable
```

---

## Notes

* Always run `--reload` (firewalld) after permanent changes if the service is running.
* Without `--permanent`, firewalld rules are lost after reboot or reload.
* ufw rules persist automatically without a separate reload step.
* Both tools support `tcp` and `udp` protocols. Specify the correct one for your service.
