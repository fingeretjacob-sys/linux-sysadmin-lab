# Linux Service Troubleshooting Lab

## Objective

Practice troubleshooting a Linux service using systemd, networking tools, and system logs.

## Environment

- Ubuntu Linux VM running in VMware Workstation
- Windows host machine
- SSH used for remote connectivity
- OpenSSH Server running on Ubuntu

## Scenario

The Ubuntu VM was accessible locally, but SSH connectivity needed to be verified and troubleshot from the Windows host.

## Troubleshooting

### 1. Verify the VM IP Address

Used:

`hostname -I`

and:

`ip addr`

This identified the IP address assigned to the Ubuntu VM.

### 2. Check the SSH Service

Used:

`systemctl status ssh`

This verifies whether the SSH service is running.

### 3. Check Port 22

Used:

`sudo ss -tulpn | grep :22`

This verifies that the SSH server is listening on TCP port 22.

### 4. Review SSH Logs

Used:

`sudo journalctl -u ssh`

Additional examples:

`sudo journalctl -u ssh -n 20`

`sudo journalctl -u ssh --since "1 hour ago"`

These commands allow SSH-related events and errors to be investigated.

### 5. Test Connectivity

From Windows PowerShell:

`ssh jacob@<Ubuntu-VM-IP>`

The SSH connection to the Ubuntu VM was successful.

## Troubleshooting Method

The troubleshooting process followed this order:

1. Verify the server is running.
2. Verify the server has an IP address.
3. Verify the SSH service is running.
4. Verify TCP port 22 is listening.
5. Review service logs.
6. Test SSH connectivity from another system.

## Skills Practiced

- Linux networking
- SSH
- systemd
- systemctl
- journalctl
- Listening ports
- Client/server troubleshooting
- Remote Linux administration
