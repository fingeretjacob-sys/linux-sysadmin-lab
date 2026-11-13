# Lab 03 - SSH and Key-Based Authentication

## Objective

Configure and verify SSH public-key authentication between a Windows workstation and an Ubuntu Linux server.

This lab demonstrates SSH networking, public/private key authentication, Linux file permissions, and remote administration.

## Environment

### SSH Client
- Windows
- OpenSSH for Windows
- ED25519 key pair

### SSH Server
- Ubuntu 24.04 LTS
- VMware Workstation
- OpenSSH Server
- Server IP: `192.168.50.130`
- SSH Port: TCP 22

## Verify SSH Server

Verified that OpenSSH Server was installed:

```bash
dpkg -l | grep openssh-server
```

Verified that SSH was listening on TCP port 22:

```bash
sudo ss -tulpn | grep :22
```

The SSH daemon was listening on both IPv4 and IPv6 interfaces.

## Network Configuration

The Ubuntu VM was configured with:

```text
Interface: ens33
IPv4 Address: 192.168.50.130/24
Network: 192.168.50.0/24
Default Gateway: 192.168.50.2
```

Network configuration was inspected using:

```bash
ip addr
ip route
```

## Create SSH Administrative Account

A dedicated Linux account was created for remote administration:

```bash
sudo useradd -m -s /bin/bash sshadmin
```

## Generate SSH Key Pair

On the Windows client, an ED25519 SSH key pair was generated:

```powershell
ssh-keygen -t ed25519 -C "linux-sysadmin-lab"
```

This created:

```text
id_ed25519      Private key
id_ed25519.pub  Public key
```

The private key remains on the client and should never be shared or committed to source control.

The public key can safely be copied to authorized SSH servers.

## Configure Public-Key Authentication

The public key was added to:

```text
/home/sshadmin/.ssh/authorized_keys
```

The SSH directory permissions were configured as:

```text
~/.ssh           700
authorized_keys  600
```

Verification showed:

```text
drwx------ sshadmin sshadmin /home/sshadmin/.ssh
-rw------- sshadmin sshadmin /home/sshadmin/.ssh/authorized_keys
```

## Connect from Windows

The Ubuntu VM was accessed remotely from Windows:

```powershell
ssh sshadmin@192.168.50.130
```

The successful connection resulted in a remote shell:

```text
sshadmin@jacob-VMware-Virtual-Platform:~$
```

## SSH Trust Model

SSH uses keys on both sides of the connection for different purposes.

### authorized_keys

Located on the Linux server:

```text
/home/sshadmin/.ssh/authorized_keys
```

This determines which client public keys are authorized to authenticate.

### known_hosts

Located on the Windows client:

```text
C:\Users\<username>\.ssh\known_hosts
```

This allows the client to recognize previously contacted SSH servers and detect unexpected changes to the server identity.

## Next Exercise

The next portion of the lab will intentionally introduce an SSH permissions problem.

The failure will be diagnosed using:

```bash
journalctl
```

and:

```powershell
ssh -v
```

The objective is to document the full troubleshooting process:

```text
Working configuration
        ↓
Introduce misconfiguration
        ↓
Authentication failure
        ↓
Investigate logs
        ↓
Identify root cause
        ↓
Apply remediation
        ↓
Verify authentication
```