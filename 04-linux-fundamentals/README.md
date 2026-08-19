# Lab 04 — Linux OS Fundamentals: Users, Privileges, Permissions \& File Structure

## Aim

To strengthen Linux operating system fundamentals required for cybersecurity by practically understanding users, groups, root privileges, `sudo`, file ownership, permissions, filesystem structure, authentication logging, and the principle of least privilege.

## Objectives

* Understand Linux users, groups, UID and GID.
* Understand the `root` user and UID 0.
* Understand the difference between the `root` user, `root` group, and `sudo` group.
* Understand how `sudo` provides temporary elevated privileges.
* Understand root shells and the difference between `sudo -i` and `su -`.
* Create and manage a normal Linux user.
* Add and remove a user from the `sudo` group.
* Understand Linux file ownership.
* Understand owner, group, and others permissions.
* Understand read, write, and execute permissions.
* Practise `chmod` using numeric permissions such as `600` and `755`.
* Practise `chown` for changing file ownership.
* Understand important Linux filesystem directories.
* Examine authentication and `sudo` activity in system logs.
* Connect Linux access control with Authentication, Authorization and Accounting (AAA).
* Apply the principle of least privilege and Defense in Depth concepts.

## Software Requirements

* VMware Workstation
* Ubuntu Server
* Terminal
* `labuser` Linux account created during the practical

## Lab Environment

|Component|Purpose|
|-|-|
|Ubuntu Server|Linux operating system used for the practical|
|`labuser`|Additional normal user used to test authorization and privileges|
|Terminal|Command execution and verification|
|`/var/log/auth.log`|Authentication and `sudo` activity review|

## Procedure

### 1\. Check Current User, UID, GID and Groups

Checked the current identity using:

```bash
whoami
id
groups
```

The `whoami` command identifies the current user, while `id` displays UID, GID and group membership.

### Evidence

!\[Figure 4.1 – Current user, UID, GID and group information](./screenshots/01-user-id-groups.png)
*Figure 4.1 – Current user, UID, GID and group information*

\---

### 2\. Understand the Root User

Checked the root account:

```bash
id root
```

Tested root-level execution through `sudo`:

```bash
sudo whoami
```

The normal user remained the logged-in user, while the `sudo` command executed with root-level privileges.

The `root` user is normally identified by UID `0`.

### Evidence

!\[Figure 4.2 – Root user identity and sudo execution](./screenshots/02-root-and-sudo.png)
*Figure 4.2 – Root user identity and sudo execution*

\---

### 3\. Test Permission Denied and sudo

A normal user attempted to create a file directly under `/`:

```bash
touch /testfile
```

The operation was denied.

The same operation was then executed with:

```bash
sudo touch /testfile
```

The operation succeeded because the command was executed with elevated privileges.

The temporary file was then removed:

```bash
sudo rm /testfile
```

### Evidence

!\[Figure 4.3 – Permission denied for a privileged operation](./screenshots/03-permission-denied.png)
*Figure 4.3 – Permission denied for a privileged operation*

!\[Figure 4.4 – Same operation successfully executed with sudo](./screenshots/04-sudo-command.png)
*Figure 4.4 – Same operation successfully executed with sudo*

\---

### 4\. Create a New User

Created a separate Linux user:

```bash
sudo adduser labuser
```

Verified the user:

```bash
id labuser
```

Switched to the new account:

```bash
su - labuser
```

Confirmed the active user:

```bash
whoami
```

The result was:

```text
labuser
```

### Evidence

!\[Figure 4.5 – Creation and verification of labuser](./screenshots/05-create-labuser.png)
*Figure 4.5 – Creation and verification of labuser*

\---

### 5\. Test and Grant sudo Authorization

Initially, `labuser` was a normal user without `sudo` authorization.

The following command was tested:

```bash
sudo whoami
```

The operation was denied.

The user was then added to the `sudo` group:

```bash
sudo usermod -aG sudo labuser
```

Group membership was verified:

```bash
groups labuser
```

After starting a new session, `labuser` could execute:

```bash
sudo whoami
```

and receive:

```text
root
```

### Evidence

!\[Figure 4.6 – labuser sudo authorization](./screenshots/06-sudo-group.png)
*Figure 4.6 – labuser before and after sudo authorization*

\---

### 6\. Remove sudo Authorization

Sudo access was also tested in reverse.

The user can be removed from the `sudo` group using:

```bash
sudo deluser labuser sudo
```

or:

```bash
sudo gpasswd -d labuser sudo
```

Membership can be checked using:

```bash
groups labuser
```

After starting a new session, `labuser` can no longer use `sudo`.

This demonstrates that authorization can be granted and revoked without deleting the user account.

\---

### 7\. Understand Root Group vs sudo Group

The practical clarified the difference between:

* `root` user
* `root` group
* `sudo` group

The `root` user is the superuser account and normally has UID `0`.

The `root` group is a Linux group and normally has GID `0`.

The `sudo` group is used by Ubuntu to authorize users to use `sudo`.

A user being a member of `sudo` does not mean the user is a member of the `root` group.

A user can technically belong to the `root` group, but group membership alone does not turn that user into the root user.

\---

### 8\. Understand Root Shells

Two methods of obtaining a root shell were discussed.

Using `sudo`:

```bash
sudo -i
```

Using the root account directly:

```bash
su -
```

The difference was established:

```text
sudo -i
    -> Uses the current user's sudo authorization.

su -
    -> Attempts to authenticate directly as the root account.
```

On a default Ubuntu installation, the root account is commonly locked, so `su -` may not work until a root password is configured.

A controlled lab experiment can configure one with:

```bash
sudo passwd root
```

The important distinction is that `sudo` is an authorization mechanism, while `su` switches to another account after authentication.

\---

### 9\. Understand File Ownership

Created a file using elevated privileges:

```bash
sudo touch ownership.txt
```

Checked ownership:

```bash
ls -l ownership.txt
```

The output can initially show:

```text
root root
```

The file owner and group were then changed:

```bash
sudo chown labuser:labuser ownership.txt
```

Verified again:

```bash
ls -l ownership.txt
```

The output then showed:

```text
labuser labuser
```

The syntax is:

```text
chown USER:GROUP FILE
```

### Evidence

!\[Figure 4.7 – File ownership before and after chown](./screenshots/07-file-ownership.png)
*Figure 4.7 – File ownership before and after chown*

\---

### 10\. Understand Owner, Group and Others

Linux file permissions are divided into three classes:

```text
OWNER       GROUP       OTHERS
  ↓           ↓           ↓
 rwx         rwx         rwx
```

For a file with:

```text
Owner = root
Group = security
```

the access decision is:

```text
root               -> owner permissions
member of security -> group permissions
everyone else      -> other permissions
```

The important rule is that Linux does not simply combine all applicable permissions. The relevant permission class is selected according to the user's relationship to the file.

A user's own group is not automatically the file's permission group. What matters is whether the user belongs to the group assigned to the file.

\---

### 11\. Understand Read, Write and Execute

A permission string such as:

```text
-rw-r--r--
```

can be divided into:

```text
-  rw-  r--  r--
   │    │    │
   │    │    └── Others
   │    └─────── Group
   └──────────── Owner
```

Permission values:

|Permission|Meaning|Value|
|-|-|-:|
|`r`|Read|4|
|`w`|Write|2|
|`x`|Execute|1|

\---

### 12\. Practise chmod 600

Created a test file:

```bash
touch security.txt
```

Changed its permissions:

```bash
chmod 600 security.txt
```

The resulting permissions are:

```text
-rw-------
```

This means:

```text
Owner  -> read + write
Group  -> no permissions
Others -> no permissions
```

Strict permissions such as `600` are particularly important for sensitive files such as private SSH keys.

### Evidence

!\[Figure 4.8 – chmod 600 and resulting permissions](./screenshots/08-chmod-600.png)
*Figure 4.8 – chmod 600 and resulting permissions*

\---

### 13\. Practise chmod 755

Created a directory:

```bash
mkdir secure-dir
```

Changed its permissions:

```bash
chmod 755 secure-dir
```

The value is calculated as:

```text
7 = 4 + 2 + 1 = rwx
5 = 4 + 1     = r-x
5 = 4 + 1     = r-x
```

Therefore:

```text
755
```

means:

```text
Owner  -> rwx
Group  -> r-x
Others -> r-x
```

The purpose was to understand the calculation rather than simply memorize `755`.

\---

### 14\. Explore the Linux Filesystem

Inspected the root filesystem:

```bash
ls /
```

Important directories were identified:

|Directory|Purpose|
|-|-|
|`/home`|Normal users' home directories|
|`/root`|Root user's home directory|
|`/etc`|System and service configuration|
|`/var/log`|System and application logs|
|`/tmp`|Temporary files|
|`/usr/bin`|Many user commands and programs|
|`/bin`|Essential commands|
|`/sbin`|System administration commands|
|`/dev`|Device interfaces|
|`/proc`|Kernel and process information|

### Evidence

!\[Figure 4.9 – Linux filesystem root directories](./screenshots/09-filesystem-structure.png)
*Figure 4.9 – Linux filesystem root directories*

\---

### 15\. Examine Authentication and sudo Logs

Authentication activity was examined using:

```bash
sudo tail -n 20 /var/log/auth.log
```

Sudo-related entries were searched using:

```bash
sudo grep "sudo" /var/log/auth.log | tail
```

The logs can contain information about authentication activity, user sessions and commands executed through `sudo`.

### Evidence

!\[Figure 4.10 – Authentication and sudo log entries](./screenshots/10-auth-log.png)
*Figure 4.10 – Authentication and sudo log entries*

\---

### 16\. Understand Logging Limitations

An important distinction was identified during the practical.

When a user runs:

```bash
sudo whoami
```

the `sudo` activity can be recorded in the authentication logs.

However, after entering a root shell using:

```bash
sudo -i
```

commands such as:

```bash
touch /root/test.txt
mkdir /root/labtest
```

are not necessarily recorded in `/var/log/auth.log` as separate `sudo` commands.

The root-shell elevation may be logged, but `auth.log` should not be treated as a complete record of every action performed inside the root shell.

More comprehensive auditing can be implemented using mechanisms such as `auditd`, depending on the system configuration.

\---

## Observations

* Linux separates normal users from the privileged `root` account.
* UID `0` identifies the root-level identity.
* A user with `sudo` access does not permanently become the root user.
* The `sudo` group and `root` group are separate concepts.
* A file contains both an owner and an owning group.
* Linux evaluates permissions using owner, group, or others.
* A user's own group does not automatically determine the file's group permissions.
* `chmod` changes permission bits, while `chown` changes ownership.
* `600` provides owner-only read/write access.
* `755` provides full permissions to the owner and read/execute permissions to group and others.
* `/etc`, `/home`, `/root`, and `/var/log` are especially important directories for cybersecurity work.
* `sudo` activity can be logged, but a sudo log is not necessarily a complete record of everything performed inside a root shell.
* User management and file permissions are practical security controls rather than only operating-system administration concepts.

## Security Concepts

### CIA Triad

**Confidentiality:** File permissions help prevent unauthorized reading.

**Integrity:** Ownership and write permissions help prevent unauthorized modification.

**Availability:** Access controls help reduce unauthorized changes that could disrupt services.

### AAA

**Authentication:** Who are you?

**Authorization:** What are you allowed to do?

**Accounting:** What did you do?

The practical demonstrated all three through user authentication, `sudo` authorization, file permissions and log inspection.

### Principle of Least Privilege

Users should receive only the privileges necessary to perform their tasks.

Using:

```bash
sudo <command>
```

instead of operating the entire session as root reduces unnecessary privilege exposure.

### Defense in Depth

The lab contributes to a layered security model:

```text
Network controls
      ↓
Authentication
      ↓
Authorization
      ↓
File permissions
      ↓
Privilege management
      ↓
Logging
      ↓
Monitoring
```

## Result

The Linux OS fundamentals practical was successfully completed. Users, groups, root privileges, `sudo`, root shells, `su`, file ownership, permissions, filesystem structure, authentication logs, AAA, least privilege, and Defense in Depth were understood and practically verified on the Ubuntu Server VM.



## Commands Used

### User and Identity

```bash
whoami
id
id root
id labuser
groups
groups labuser
```

### User Management

```bash
sudo adduser labuser
sudo usermod -aG sudo labuser
sudo deluser labuser sudo
sudo gpasswd -d labuser sudo
```

### Privilege Management

```bash
sudo whoami
sudo -i
su -
sudo passwd root
```

### Files and Directories

```bash
touch test.txt
mkdir secure-dir
ls
ls -l
ls -ld secure-dir
pwd
```

### Permissions

```bash
chmod 600 security.txt
chmod 755 secure-dir
```

### Ownership

```bash
sudo chown labuser:labuser ownership.txt
```

### Filesystem

```bash
ls /
```

### Logs

```bash
sudo tail -n 20 /var/log/auth.log
sudo grep "sudo" /var/log/auth.log | tail
```

## Evidence

!\[Figure 4.1 – Current user, UID, GID and group information](./screenshots/01-user-id-groups.png)
*Figure 4.1 – Current user, UID, GID and group information*

!\[Figure 4.2 – Root user identity and sudo execution](./screenshots/02-root-and-sudo.png)
*Figure 4.2 – Root user identity and sudo execution*

!\[Figure 4.3 – Permission denied for a privileged operation](./screenshots/03-permission-denied.png)
*Figure 4.3 – Permission denied for a privileged operation*

!\[Figure 4.4 – Same operation successfully executed with sudo](./screenshots/04-sudo-command.png)
*Figure 4.4 – Same operation successfully executed with sudo*

!\[Figure 4.5 – Creation and verification of labuser](./screenshots/05-create-labuser.png)
*Figure 4.5 – Creation and verification of labuser*

!\[Figure 4.6 – labuser sudo authorization](./screenshots/06-sudo-group.png)
*Figure 4.6 – labuser before and after sudo authorization*

!\[Figure 4.7 – File ownership before and after chown](./screenshots/07-file-ownership.png)
*Figure 4.7 – File ownership before and after chown*

!\[Figure 4.8 – chmod 600 and resulting permissions](./screenshots/08-chmod-600.png)
*Figure 4.8 – chmod 600 and resulting permissions*

!\[Figure 4.9 – Linux filesystem root directories](./screenshots/09-filesystem-structure.png)
*Figure 4.9 – Linux filesystem root directories*

!\[Figure 4.10 – Authentication and sudo log entries](./screenshots/10-auth-log.png)
*Figure 4.10 – Authentication and sudo log entries*

