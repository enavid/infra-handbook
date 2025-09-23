# Installing Docker CE on Oracle Linux 7.9

This document provides a professional step-by-step guide to install Docker CE on Oracle Linux 7.9.

---

## 1. Remove Old Oracle Docker Packages

Oracle Linux ships with its own Docker packages which conflict with Docker CE. These must be removed before installation:

```bash
sudo yum remove -y docker docker-common docker-selinux docker-engine docker-client docker-cli
rpm -qa | grep docker
```

Ensure no `docker-cli` or `docker-engine` packages remain installed.

---

## 2. Update System and Install Required Tools

```bash
sudo yum update -y
sudo yum install -y yum-utils
```

---

## 3. Prerequisites

Ensure the required repositories are enabled: `ol7_addons`, `ol7_optional_latest`, and optionally `ol7_developer`.

```bash
sudo yum-config-manager --enable ol7_addons
sudo yum-config-manager --enable ol7_optional_latest
sudo yum install -y oraclelinux-developer-release-el7
sudo yum-config-manager --enable ol7_developer:contentReference[oaicite:2]{index=2}
```

---

## 4. Configure Docker Repository

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

---

## 5. Install Docker CE

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 6. Enable and Start Docker Service

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 7. Configure firewall

```bash
sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0
sudo firewall-cmd --permanent --zone=trusted --add-masquerade
```

---

## 8. Optional: Run Docker Without Sudo

```bash
sudo groupadd docker   # Only if group does not exist
sudo usermod -aG docker <your-username>
newgrp docker
```

---

## 7. Verify Docker Installation

```bash
docker run hello-world
```

---

## Notes

- Ensure SELinux and firewalld do not block Docker.
- Oracle Linux’s UEK kernel is recommended for compatibility.
- For enterprise environments, review Docker’s official documentation and Oracle Linux release notes.
