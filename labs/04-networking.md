# Linux Service Troubleshooting Lab

## Scenario

SSH connections to an Ubuntu server were unavailable.

## Investigation

Checked system uptime:

`uptime`

Checked network configuration:

`ip addr`

Checked listening ports:

`ss -tulpn | grep :22`

Checked SSH service:

`systemctl status ssh`

Reviewed SSH logs:

`journalctl -u ssh`

## Root Cause

The SSH service was stopped.

## Resolution

Started the service:

`sudo systemctl start ssh`

Verified SSH was listening on TCP port 22.

Tested SSH connectivity from the Windows host.