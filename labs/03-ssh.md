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


## SSH Authentication Troubleshooting

To practice troubleshooting SSH authentication, I intentionally changed the permissions on the `authorized_keys` file from `600` to `666`.

```bash
sudo chmod 666 /home/sshadmin/.ssh/authorized_keys
```

This changed the permissions from:

```text
-rw-------
```

to:

```text
-rw-rw-rw-
```

The file was now writable by the owner, group, and other users.

### Client-Side Investigation

From the Windows SSH client, I enabled verbose logging:

```powershell
ssh -v sshadmin@192.168.50.130
```

The SSH client successfully located and offered the ED25519 private key:

```text
Offering public key: C:\Users\romot\.ssh\id_ed25519
```

However, the server rejected public-key authentication and the client fell back to password authentication.

This showed that the client had located the correct private key, but the SSH server was refusing the corresponding public key.

### Server-Side Investigation

On the Ubuntu server, I reviewed the SSH service logs:

```bash
sudo journalctl -u ssh --since "10 minutes ago" --no-pager
```

The SSH daemon reported:

```text
Authentication refused: bad ownership or modes for file /home/sshadmin/.ssh/authorized_keys
```

### Root Cause

The `authorized_keys` file had overly permissive permissions.

SSH rejected the file because allowing other users to modify `authorized_keys` could allow an unauthorized user to add their own public key and gain access to the account.

### Resolution

I restored the file permissions:

```bash
sudo chmod 600 /home/sshadmin/.ssh/authorized_keys
```

The secure configuration was:

```text
~/.ssh           700
authorized_keys  600
```

### Verification

I reconnected from Windows using verbose SSH:

```powershell
ssh -v sshadmin@192.168.50.130
```

After restoring the permissions, public-key authentication succeeded and the remote Ubuntu shell was opened successfully.

## Troubleshooting Methodology

This exercise demonstrated a structured troubleshooting process:

```text
SSH authentication failure
        ↓
Inspect client with ssh -v
        ↓
Verify correct key is offered
        ↓
Inspect server logs with journalctl
        ↓
Identify insecure authorized_keys permissions
        ↓
Restore permissions
        ↓
Retest SSH authentication
        ↓
Successful remote login
```

## What I Learned

This lab demonstrated:

- How SSH operates over TCP port 22
- The relationship between SSH clients and servers
- Public and private SSH keys
- The purpose of `authorized_keys`
- The purpose of `known_hosts`
- Secure permissions for SSH files
- Using `ssh -v` for client-side troubleshooting
- Using `journalctl` for server-side troubleshooting
- How Linux file permissions can affect authentication
- How to validate a fix by reproducing the original connection