# Lab 01 - Linux Users and Groups

## Objective

Create multiple Linux user accounts and configure a shared administrative group to control access to a shared directory.

## Environment

- Ubuntu Linux
- VMware Workstation
- Bash
- Linux user and group management utilities

## Create Administrative Group

Created a group named `labadmins`:

```bash
sudo groupadd labadmins

Verified the group:

Shell
getent group labadmins
Create Test Users

Created two Linux users with home directories and Bash as their login shell:

Shell
sudo useradd -m -s /bin/bash labadmin1
sudo useradd -m -s /bin/bash labadmin2
 

Verified the accounts:

Shell
getent passwd labadmin1
getent passwd labadmin2
Add Users to Administrative Group

Added both accounts to the labadmins supplementary group:

Shell
sudo usermod -aG labadmins labadmin1
sudo usermod -aG labadmins labadmin2

Verified group membership:

Shell
id labadmin1
id labadmin2

Both users retained their individual primary groups while receiving membership in the labadmins supplementary group.

Create Shared Administrative Directory

Created a shared directory:

Shell
sudo mkdir -p /srv/itshare

Assigned the labadmins group:

Shell
sudo chown root:labadmins /srv/itshare

Configured permissions:

Shell
sudo chmod 2770 /srv/itshare

The permissions provide:

Full access for the owner
Full access for members of labadmins
No access for other users
Group inheritance using the setgid bit
Test Access

Created files as each administrative user:

Shell
sudo -u labadmin1 touch /srv/itshare/test-from-admin1.txt
sudo -u labadmin2 touch /srv/itshare/test-from-admin2.txt

The resulting files inherited the labadmins group:

Plain Text
-rw-rw-r-- labadmin1 labadmins test-from-admin1.txt
-rw-rw-r-- labadmin2 labadmins test-from-admin2.txt

An account outside the labadmins group received Permission denied when attempting to list the directory, confirming that the access restrictions were functioning correctly.

What I Learned

This lab demonstrated the difference between primary and supplementary Linux groups and how group membership can be used to control access to shared resources.

I also used the setgid permission on a shared directory so that files created by different users automatically inherit the same group ownership. This provides a practical method for managing collaborative directories without granting access to unrelated users.