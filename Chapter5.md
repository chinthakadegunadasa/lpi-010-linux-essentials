# Chapter 5: Security and File Permissions
Data security, access control, and identity management are fundamental pillars of enterprise Linux administration. The Linux security model relies on multi-layered defenses: POSIX file permissions, explicit user/group identities, access control lists, privilege escalation boundaries, and kernel-level mandatory access control systems.
In this chapter, you will master POSIX file permissions, advanced permission attributes (SUID, SGID, Sticky Bit), explicit Access Control Lists (ACLs), user account administration, and privilege management via sudo.
## 5.1 POSIX File Permissions and Ownership
### The POSIX Security Model
Every file and directory in Linux is owned by a specific **User** (u) and **Group** (g), and carries access definitions for **Others** (o).
File permissions are divided into three core access classes:
 * **Read (r / 4):** Grants permission to read file contents or list directory entries.
 * **Write (w / 2):** Grants permission to modify file contents or create/delete files inside a directory.
 * **Execute (x / 1):** Grants permission to execute a binary/script or traverse (cd) into a directory.
```
       File Type (- = regular, d = directory, l = symlink)
       │
       │  User (Owner)    Group          Others
       │  ┌───┴───┐      ┌───┴───┐      ┌───┴───┐
       -  r   w   x      r   -   x      r   -   -
       │  │   │   │      │   │   │      │   │   │
       │  4 + 2 + 1      4 + 0 + 1      4 + 0 + 0
       │  =========      =========      =========
       │    Mode 7         Mode 5         Mode 4  ==> Numeric Mode: 754

```
### Symbolic vs. Octal Permission Notation
Permissions can be modified using either **octal values** or **symbolic representation**:
 * **Octal Notation:** Summing binary values for each group (r=4, w=2, x=1):
   * 7 (4+2+1): Read, write, and execute.
   * 6 (4+2+0): Read and write.
   * 5 (4+0+1): Read and execute.
   * 4 (4+0+0): Read-only.
 * **Symbolic Notation:** Modifying specific targets using operations (+, -, =):
   * chmod u+x script.sh : Adds execute permission for the file owner.
   * chmod g-w file.txt : Removes write permission from the group.
   * chmod o=r file.txt : Sets others permission explicitly to read-only.
### Managing Ownership and Default Creation Modes (umask)
 * **chown:** Changes file owner and group (chown user:group file).
 * **chgrp:** Changes group ownership (chgrp group file).
 * **umask:** Defines default permissions masked out during new file/directory creation.
   * Default maximum file mode: 666 (rw-rw-rw-)
   * Default maximum directory mode: 777 (rwxrwxrwx)
   * A standard server umask of 022 results in default file mode 644 (rw-r--r--) and directory mode 755 (rwxr-xr-x).
### Hands-On Lab 5.1: Managing Basic Permissions and Default Creation Modes
In this lab, you will assign file permissions, alter ownership, and observe custom umask masks.
**Step 1: Inspect baseline permission modes.**
```bash
touch sample_file.txt
ls -l sample_file.txt

```
**Step 2: Modify permissions using octal and symbolic modes.**
```bash
chmod 640 sample_file.txt
ls -l sample_file.txt

chmod u+x,g+w sample_file.txt
ls -l sample_file.txt

```
**Step 3: Test temporary umask adjustments.**
```bash
umask 027
touch secure_file.txt
mkdir secure_dir
ls -ld secure_file.txt secure_dir

```
*Verification:* secure_file.txt receives mode 640 (rw-r-----), and secure_dir receives mode 750 (rwxr-x---).
## 5.2 Special Permission Bits & Access Control Lists (ACLs)
### SUID, SGID, and the Sticky Bit
Standard POSIX bits are augmented by special attributes for specialized binary and shared directory workflows:
| Special Bit | Octal Value | Target | Behavior on Execution / Creation |
|---|---|---|---|
| **SUID** (Set Owner User ID) | 4000 | Executable File | Process executes with the privileges of the file *owner*, not the executing user (e.g., /usr/bin/passwd). |
| **SGID** (Set Group ID) | 2000 | Directory / Binary | Files created inside the directory inherit the directory's *group ownership*, rather than the primary group of the user creating the file. |
| **Sticky Bit** | 1000 | Directory | Users can only delete or rename files inside the directory if they are the file owner or root (e.g., /tmp). |
Assigning special permissions:
 * chmod 4755 binary or chmod u+s binary (SUID)
 * chmod 2775 directory or chmod g+s directory (SGID)
 * chmod 1777 directory or chmod +t directory (Sticky Bit)
### POSIX Access Control Lists (ACLs)
Standard POSIX permissions only accommodate one user and one group per file. **POSIX ACLs** extend access controls to assign explicit permissions for arbitrary users or groups:
 * **getfacl:** Displays file/directory ACL entries.
 * **setfacl:** Assigns or removes ACL entries:
   * Grant user explicit read/write access: setfacl -m u:alice:rw file.txt
   * Grant group explicit access: setfacl -m g:developers:rx file.txt
   * Set default inheritance rules for subdirectories: setfacl -d -m g:developers:rwx shared_dir/
   * Remove single entry: setfacl -x u:alice file.txt
   * Remove all extended ACLs: setfacl -b file.txt
### Hands-On Lab 5.2: Configuring Shared Directories and Extended ACLs
In this lab, you will configure a multi-user collaborative directory with SGID, Sticky Bits, and fine-grained POSIX ACLs.
**Step 1: Create a shared group directory with SGID enabled.**
```bash
sudo mkdir -p /srv/shared_project
sudo groupadd project_team
sudo chown :project_team /srv/shared_project
sudo chmod 2770 /srv/shared_project
ls -ld /srv/shared_project

```
**Step 2: Assign specific extended user permissions via POSIX ACLs.**
```bash
sudo setfacl -m u:nobody:rx /srv/shared_project
sudo getfacl /srv/shared_project

```
**Step 3: Define default inherited ACL rules for all newly created sub-items.**
```bash
sudo setfacl -d -m g:project_team:rwx /srv/shared_project
sudo getfacl /srv/shared_project

```
## 5.3 User Accounts, Group Administration, and Authentication
### User and Group Database Structure
Linux manages local identity through core configuration files:
```
/etc/passwd          ──► Account Metadata (Username, UID, GID, Home Dir, Shell)
/etc/shadow          ──► Encrypted Passwords & Expiration Details (Root Readable Only)
/etc/group           ──► Group Membership Registers
/etc/gshadow         ──► Encrypted Group Account Authentication Details
/etc/default/useradd ──► Baseline User Provisioning Defaults
/etc/skel/           ──► Skeleton Templates Copied to New Home Directories

```
### Account Management Operations
Core user management tools:
 * **useradd:** Creates new user accounts (useradd -m -s /bin/bash username).
 * **usermod:** Modifies existing account properties (usermod -aG groupname username appends user to secondary group).
 * **userdel:** Removes user accounts (userdel -r username removes user and home directory).
 * **groupadd / groupdel:** Creates or removes local groups.
 * **passwd:** Updates account passwords (passwd username).
 * **chage:** Manages password expiration aging policies (chage -l username).
### Hands-On Lab 5.3: Provisioning Users and Enforcing Aging Policies
In this lab, you will provision an enterprise user account, configure secondary groups, and enforce password security policies.
**Step 1: Create a new user account with home directory setup.**
```bash
sudo useradd -m -s /bin/bash -c "System Specialist" sysadmin_dev
id sysadmin_dev

```
**Step 2: Assign secondary group memberships.**
```bash
sudo usermod -aG project_team sysadmin_dev
id sysadmin_dev

```
**Step 3: Enforce account expiration and password aging policies.**
```bash
sudo chage -M 90 -W 14 sysadmin_dev
sudo chage -l sysadmin_dev

```
## 5.4 Privilege Escalation Frameworks (sudo)
### The sudo Control Framework
Directly logging in as the root superuser (root) exposes systems to untracked operational risks. The **sudo** (Superuser Do) framework grants delegated command privileges to authenticated normal users while logging every executed command to /var/log/auth.log.
### Configuring /etc/sudoers
The primary privilege specification file is /etc/sudoers, edited exclusively using the **visudo** tool to perform syntax checks prior to saving.
```
# Privilege Specification Anatomy:
# User / Group     Host = (RunAs_User : RunAs_Group)     Commands

root               ALL=(ALL:ALL)                         ALL
%sudo              ALL=(ALL:ALL)                         ALL
alice              ALL=(ALL)                             NOPASSWD: /bin/systemctl restart nginx

```
 * **%group:** The % prefix indicates a system group rule.
 * **NOPASSWD::** Allows execution of specific administrative binaries without re-prompting for user credentials.
Custom sudo rules should be placed in individual files inside /etc/sudoers.d/ rather than modifying the main /etc/sudoers file directly.
### Hands-On Lab 5.4: Configuring Fine-Grained Delegated Privileges
In this lab, you will configure scoped administrative delegation using drop-in sudoers files.
**Step 1: Create a scoped drop-in configuration file using visudo.**
```bash
sudo visudo -f /etc/sudoers.d/sysadmin_dev

```
**Step 2: Add restricted service management rights.**
Insert the following privilege rule and save:
```text
sysadmin_dev ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart ssh, /usr/bin/systemctl status ssh

```
**Step 3: Verify privileges from the user context.**
```bash
sudo -u sysadmin_dev sudo -l

```
## Chapter 5 Summary Checklist
Before moving on to **Chapter 6: Software Package Management**, make sure you can complete these tasks:
 * [ ] Can you calculate and assign POSIX permissions using octal (chmod 755) and symbolic (chmod u+x) syntax?
 * [ ] Do you know how default file permissions are calculated using umask?
 * [ ] Can you explain the roles of SUID, SGID, and Sticky Bit attributes in multi-user setups?
 * [ ] Do you know how to grant fine-grained permissions using POSIX ACLs (setfacl and getfacl)?
 * [ ] Can you safely configure delegated administrative privileges using visudo and /etc/sudoers.d/ drop-in files?
