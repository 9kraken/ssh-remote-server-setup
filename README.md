# SSH Remote Server Setup

Setup of a remote Linux server and SSH configuration for secure access.

https://roadmap.sh/projects/ssh-remote-server-setup

## Overview

The goal of this project was to provision a remote VPS, configure secure user access using SSH keys, and implement basic security measures like fail2ban.

## Key Features
* **Non-root Access**: Created a dedicated `dev` user to avoid working as root.
* **Key-based Authentication**: Configured SSH keys for secure, passwordless login.
* **SSH Config Aliases**: Simplified connection commands using `~/.ssh/config`.
* **Security**: Installed and configured `fail2ban` to protect against brute-force attacks.

# Implementation Details

## 1. Initial Access and User Setup

After renting the VPS, I accessed it via the root account to create a new administrative user:
```bash
ssh root@<server_ip>
adduser dev
usermod -aG sudo dev
```

## 2. SSH Key Configuration

I generated a pair of SSH keys and added the public keys to the server to allow access for the `dev` user:

* Path on server: `/home/dev/.ssh/authorized_keys`

* Setup:
```bash
mkdir -p /home/dev/.ssh
# Added 2 public keys to authorized_keys
chmod 700 /home/dev/.ssh
chmod 600 /home/dev/.ssh/authorized_keys
```

Now, the server can be accessed via `ssh -i <path_to_private_key> dev@<server_ip>`

## 3. SSH Client Optimization

To simplify the connection process, I configured aliases in my local `~/.ssh/config file`:
```text
Host vps1
    HostName <server_ip>
    User dev
    IdentityFile ~/.ssh/vps_key1
    IdentitiesOnly yes
    ServerAliveInterval 60

Host vps2
    HostName <your_server_ip>
    User dev
    IdentityFile ~/.ssh/vps_key2
    IdentitiesOnly yes
    ServerAliveInterval 60
```

Use the `IdentitiesOnly` `yes` flag to force SSH to use specific keys for different accounts on the same host.

## 4. Server Security with Fail2Ban

To prevent brute-force attacks on the SSH port, I installed `fail2ban`:
```bash
sudo apt update
sudo apt install fail2ban

sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

To configure `fail2ban` we need to create SSH config:
```bash
# open config
sudo nano /etc/fail2ban/jail.local
```
in config file
```text
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
findtime = 10m
bantime = 10m
ignoreip = 127.0.0.1/8 ::1   # Do not ban yourself
```
apply config by restarting fail2ban service
```bash
sudo systemctl restart fail2ban
```
## 5. Security Hardening (optional)

You can disable password login to ensure only users with authorized SSH keys can access the system.  
To do this, change or add next lines in `/etc/ssh/sshd_config` (on remote server):
```bash
PermitRootLogin no                  # forbid root login
PasswordAuthentication no           # for all users (root, dev, ...)
PubkeyAuthentication yes            # allow only keys
ChallengeResponseAuthentication no  
```
**Note**: Check `/etc/ssh/sshd_config.d/` for any config snippets that might override global settings.  
The `sshd` daemon prioritizes the first value it encounters for a parameter.
