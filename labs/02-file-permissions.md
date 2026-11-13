# Lab 02 - Linux File Permissions

## Objective

Practice Linux file and directory permissions by creating a shared administrative directory, configuring ownership, testing access between users, troubleshooting a permission issue, and implementing group inheritance.

## Environment

- Ubuntu Linux
- VMware Workstation
- Bash
- Linux users: `labadmin1` and `labadmin2`
- Shared group: `labadmins`

## Understanding Linux Permissions

Linux permissions are divided into three categories:

| Permission | Value | Meaning |
|---|---:|---|
| Read (`r`) | 4 | View file contents |
| Write (`w`) | 2 | Modify a file |
| Execute (`x`) | 1 | Execute a file or traverse a directory |

For example:

```text
770 = rwxrwx---
640 = rw-r-----
```

A permission of `640` gives:

- Owner: read and write
- Group: read
- Others: no access

## Create the Permission Lab

Created a shared directory:

```bash
sudo mkdir -p /srv/permission-lab
```

Assigned ownership to `labadmin1` and the `labadmins` group:

```bash
sudo chown labadmin1:labadmins /srv/permission-lab
```

Configured the directory with `770` permissions:

```bash
sudo chmod 770 /srv/permission-lab
```

This allowed the owner and members of `labadmins` full access while preventing access by other users.

## Create a Configuration File

Created a file as `labadmin1`:

```bash
sudo -u labadmin1 touch /srv/permission-lab/server-config.txt
```

Configured the file with `640` permissions:

```bash
sudo chmod 640 /srv/permission-lab/server-config.txt
```

The intended access model was:

```text
Owner:   rw-
Group:   r--
Others:  ---
```

This should allow `labadmin1` to modify the file and members of the shared group to read it.

## Permission Issue

When `labadmin2` attempted to read the file:

```bash
sudo -u labadmin2 cat /srv/permission-lab/server-config.txt
```

Linux returned:

```text
Permission denied
```

Inspection of the file revealed:

```text
-rw-r----- 1 labadmin1 labadmin1 server-config.txt
```

Although the permissions were set correctly to `640`, the file belonged to the `labadmin1` group instead of the shared `labadmins` group.

Because `labadmin2` was a member of `labadmins`, not the private `labadmin1` group, the group read permission did not apply.

## Resolution

Changed the file's group ownership:

```bash
sudo chown labadmin1:labadmins /srv/permission-lab/server-config.txt
```

This allowed `labadmin2` to use the group's read permission.

I then tested whether `labadmin2` could modify the file:

```bash
sudo -u labadmin2 bash -c 'echo "Modified by labadmin2" >> /srv/permission-lab/server-config.txt'
```

The operation returned:

```text
Permission denied
```

This was expected because `640` grants the group read access but not write access.

## Preventing the Issue

To make files created inside the shared directory automatically inherit the `labadmins` group, I enabled the setgid bit:

```bash
sudo chmod 2770 /srv/permission-lab
```

Verification:

```text
drwxrws--- 2 labadmin1 labadmins /srv/permission-lab
```

The `s` in the group execute position indicates that setgid is enabled.

Files and subdirectories created inside this directory can now inherit the `labadmins` group, providing more consistent group ownership for shared administrative resources.

## What I Learned

This lab demonstrated that Linux access control depends on both permissions and ownership.

A file can have the correct numeric permissions but still deny access if its user or group ownership is incorrect.

I practiced:

- Reading symbolic Linux permissions
- Converting permissions to octal values such as `640`, `770`, and `2770`
- Using `chmod` to modify permissions
- Using `chown` to modify user and group ownership
- Troubleshooting `Permission denied`
- Understanding primary versus supplementary groups
- Using setgid to maintain group ownership in shared directories

The key troubleshooting lesson was to check both **permissions and ownership** rather than assuming a `Permission denied` error is caused only by `chmod`.