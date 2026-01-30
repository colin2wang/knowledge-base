# Ubuntu

## Package Repository Configuration

> Backup current sources list before modification.
```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo nano /etc/apt/sources.list
```

> Replace with one of the following mirror configurations:

### Ubuntu 22.04 LTS (Jammy Jellyfish) - Recommended

```
# Tsinghua University Mirror
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# Aliyun Mirror
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse

# USTC Mirror
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 163 Mirror
deb http://mirrors.163.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ jammy-backports main restricted universe multiverse
```

### Ubuntu 20.04 LTS (Focal Fossa) - Legacy Support

```
# Tsinghua University Mirror
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
```

> After updating the sources list, run:
```shell
sudo apt update
sudo apt upgrade
```

## Hostname Management

### Check Current Hostname

There are several ways to quickly check the hostname in Ubuntu system:

1. Open a GNOME terminal window and observe the command prompt - the hostname is typically displayed after the "@" symbol
2. Run either of these commands in terminal:
```shell
hostname
uname -n
```

### Temporary Hostname Change

To temporarily change the hostname, run:
```shell
sudo hostname new-hostname
```

Where "new-hostname" can be any valid string. Note that this method doesn't persist after reboot - the system will revert to the original hostname.

Example:
```shell
sudo hostname ubuntu-temp
```

The new hostname won't immediately appear in the current terminal session. You'll need to open a new terminal window (SSH connections need to be re-established).

### Permanent Hostname Change

To permanently change the hostname in Ubuntu:

1. Edit the hostname file:
```shell
sudo nano /etc/hostname
```

2. Replace the existing hostname with your desired name and save the file

3. Also update the hosts file to maintain consistency:
```shell
sudo nano /etc/hosts
```

4. Find the line containing the current hostname and update it:
```
127.0.1.1       old-hostname
```
to:
```
127.0.1.1       new-hostname
```

5. Apply changes without reboot:
```shell
sudo hostnamectl set-hostname new-hostname
sudo systemctl restart systemd-logind
```

### Difference Between /etc/hostname and /etc/hosts

- `/etc/hostname`: Contains the actual system hostname
  Example content: `ubuntu-server`

- `/etc/hosts`: Contains IP address to hostname mappings
  The hostname mapping here is separate from the system hostname
  Example content:
  ```
  127.0.0.1       localhost
  127.0.1.1       ubuntu-server
  ```

Note: Different Linux distributions store hostname information in different locations. For example, Fedora stores it in `/etc/sysconfig/network`. Always verify the correct location for your specific distribution.


## Time Zone and Clock Synchronization

### Fix 8-Hour Time Difference Issue

If your Ubuntu system shows a time difference of 8 hours (common in dual-boot systems with Windows):

```shell
# Set hardware clock to local timezone (Windows compatibility)
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

### Modern Time Management Commands

```shell
# Check current time settings
timedatectl status

# List available timezones
timedatectl list-timezones

# Set timezone (example for Shanghai)
sudo timedatectl set-timezone Asia/Shanghai

# Enable automatic time synchronization
sudo timedatectl set-ntp true

# Manual time synchronization
sudo systemctl restart systemd-timesyncd
```

### Dual-Boot System Considerations

For systems dual-booting with Windows, you have two options:

1. **Hardware clock in local time** (recommended for dual-boot):
```shell
sudo timedatectl set-local-rtc 1
```

2. **Hardware clock in UTC** (Linux standard):
```shell
sudo timedatectl set-local-rtc 0
```

Choose based on whether you primarily use Linux or need Windows compatibility.
