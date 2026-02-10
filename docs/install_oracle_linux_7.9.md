
# Oracle Linux 7.9 Installation Guide

## 1. Download ISO

Download the Oracle Linux 7.9 ISO from the official repository:

```
https://yum.oracle.com/ISOS/OracleLinux/OL7/u9/x86_64/OracleLinux-R7-U9-Server-x86_64-dvd.iso
```

## 2. Partitioning Scheme

Use **LVM** as the file system type with **XFS** format.

### For Application Server:

| Mount Point | Size            | Type               |
| ----------- | --------------- | ------------------ |
| `/boot`   | 1 GB            | Standard Partition |
| `swap`    | 16 GB           | LVM                |
| `/var`    | 30 GB           | LVM                |
| `/`       | Remaining space | LVM                |

### For Database Server:

| Mount Point | Size  | Type                              |
| ----------- | ----- | --------------------------------- |
| `swap`    | 16 GB | LVM                               |
| `/`       | 40 GB | LVM                               |
| `/u01`    | 70 GB | LVM (mount after OS installation) |
| `/u02`    | 70 GB | LVM (mount after OS installation) |

## 3. Creating and Mounting u01 and u02 Partitions

For database servers, `/u01` and `/u02` partitions should be created and mounted after the initial OS installation.

### Steps to create and mount u01 and u02:

1. **Create the logical volumes:**

```bash
# Create logical volume for u01
lvcreate -L 70G -n lv_u01 vg_name

# Create logical volume for u02
lvcreate -L 70G -n lv_u02 vg_name
```

2. **Format the logical volumes with XFS:**

```bash
mkfs.xfs /dev/vg_name/lv_u01
mkfs.xfs /dev/vg_name/lv_u02
```

3. **Create mount points:**

```bash
mkdir /u01
mkdir /u02
```

4. **Mount the partitions:**

```bash
mount /dev/vg_name/lv_u01 /u01
mount /dev/vg_name/lv_u02 /u02
```

5. **Configure automatic mounting on boot:**

Edit the `/etc/fstab` file to ensure these partitions mount automatically after every restart:

```bash
vi /etc/fstab
```

Add the following lines:

```
/dev/vg_name/lv_u01    /u01    xfs    defaults    0 0
/dev/vg_name/lv_u02    /u02    xfs    defaults    0 0
```

6. **Verify the configuration:**

```bash
# Test fstab entries
mount -a

# Verify mounted filesystems
df -h
```

## 4. Software Selection During Installation

During the installation process, select **Infrastructure Server** as the base environment and check the following additional packages:

1. Debugging Tools
2. Hardware Monitoring Utilities
3. Java Platform
4. Large Systems Performance
5. Performance Tools
6. Compatibility Libraries
7. Development Tools
8. Security Tools
9. System Administration Tools

## 5. Network Configuration (Static IP)

To configure a static IP address on Oracle Linux 7.9:

1. **Edit the network configuration file:**

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

2. **Add/modify the following configuration:**

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=194.180.11.114
PREFIX=24
GATEWAY=194.180.11.1
DNS1=1.1.1.1
DNS2=8.8.8.8
```

3. **Restart the NetworkManager service:**

```bash
systemctl restart NetworkManager
```

4. **Verify the network configuration:**

```bash
ip addr show
ping -c 4 8.8.8.8
```

## Post-Installation Notes

* Ensure SELinux is configured according to your security requirements
* Update the system after installation: `yum update -y`
* Configure firewall rules as needed
* For database servers, verify that `/u01` and `/u02` are properly mounted after reboot

---

**Note:** Replace `vg_name` with your actual volume group name when creating logical volumes for u01 and u02.
