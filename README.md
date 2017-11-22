# Level 5 Workstation User Guide

## About This Document

**Authored by:**

Brad Frank & Emily Lawrence  
Harvard-MIT Data Center

This document instructs administrators and staff on using the level 5 workstations with regard to proper handling and processing of data, and user management.

**Note: When substituting for placeholders, i.e. [variable], make sure not to include the brackets.**

## Administration

### A.1. Users & Groups

Administrators belong to a built-in system group called wheel. The group has `sudo` privileges and can execute commands that normal users cannot. Never add a user to this group unless they have been properly cleared by EdLabs processes for administrator rights. See Section [A.1.3](#a13-addingdeleting-an-administrator) to grant administrator privileges to a user.

All EdLabs users should be joined to the general `edlabs` group. This provides access to all datasets, unless a dataset has been specifically assigned to a different group. A dataset will be assigned to a different group in the event that access needs to be restricted to a subset of users.

For stricter access to a dataset, a new group should be created. A good practice is to use the same name as the data for the group name, prefixed with "edlabs". For example, a dataset `/mnt/edlabs_data/govdata` would belong to a group `edlabs_govdata`. For setting up a new dataset, see Section [A.2.1.](#a21-creating-a-new-project).

#### A.1.1. Group Commands

##### A.1.1.1. Creating A New Group

Use a descriptive name for the group; it's recommended that if you're adding a new project, to use the same name of the project directory for the group. _Groups can include numbers, but avoid spaces and symbols._

`sudo groupadd [group]`

##### A.1.1.2. Adding An Existing User To A Group

To add a user that already exists to a group:  
`sudo usermod -aG [group] [username]`

To add the existing user to multiple groups at once, comma separate (no space) the group names:  
`sudo usermod -aG [group],[group] [username]`

##### A.1.1.3. Removing A User From A Group

To remove a user from a group:  
`sudo gpasswd -d [username] [group]`

If making a new user an administrator, see Section [A.1.3](#a13-addingdeleting-an-administrator).

#### A.1.2. New User Account Procedure

All new user accounts must be approved by an EdLabs administrator. Account usernames should follow the pattern first initial plus last name (e.g. "Pat Smith" becomes "psmith").

By default, all accounts must expire in one year. If the staff needs access beyond that, an administrator must extend the account. Both the administrator and user will receive notices beginning at 30 days before expiration, after which the account will become locked. The account can be extended at any point before or after expiration.

Account passwords do not expire once set, although users can reset his or her password at any time (`passwd` from the terminal); if an administrator needs to reset a user’s password, follow the steps in Section [A.1.2.3](#a123-resetting-the-users-password).

##### A.1.2.1. Create a New EdLabs User

To create a new user belonging to the edlabs group:  
`sudo useradd –G edlabs [username]`

The user can also be added to additional groups, which are comma separated (no spaces):  
`sudo useradd –G [group],[group] [username]`

If the user is not added to a group at this time, see Section [A.1.1.2.](#a112-adding-an-existing-user-to-a-group) for adding a group membership later.

##### A.1.2.2. Account Expiration

Set the account to expire after one year. _This step must not be skipped, and must be applied to all accounts._  
`extend [username]`

**Note:** The `extend` command is a global alias:  
`` `sudo chage -E `date -d "1 year" +%Y-%m-%d` [username]` ``

##### A.1.2.3. Resetting The User's Password

All new users must change their password on first log in. These commands set a temporary password that is safe to give to the user, and expires the password which forces a reset.

1. `sudo mkpasswd [username]`
2. `frpasswd [username]`

**Note:** The frpasswd command is a global alias:  
`sudo chage -d 0 [username]`

After creating the account, the new user should log in; they will be prompted to reset their password. Once the desktop environment has loaded, click through the intro screens, then the user should configure Thunderbird to receive system mail, see Section [B.1.2.1.](#b121-configuring-thunderbird)

#### A.1.3. Adding/Deleting an Administrator

##### A.1.3.1. Using The "wheel" Group

The "wheel" group is a built-in group that has administrator privileges.

* To add a user to the admin group: `sudo usermod -aG wheel [username]`
* To remove a user from the admin group: `sudo gpasswd -d [username] wheel`

##### A.1.1.2. Deliver Admin Mail to New User

A new administrator should also get a copy of the mail that is sent to the root user. This functionality is a requirement of level 5 machines.

1. Add the user to aliases file:
    1. Open the aliases config file for editing: `sudo gedit /etc/aliases`
    2. Add a new alias entry (usually bottom of file: "Person who should get root’s mail") or remove the appropriate username. When adding additional usernames, separate with a comma and space ([username], [username]). You're looking for the line `root: [username]`.
    3. Save and close the file.
    4. Commit the change: `sudo newaliases`
2. Modify root’s mail forwarding:
    1. `sudo gedit /root/.forward`
    2. Add or remove users as necessary. Multiple accounts are separated by only spaces. The format is `[username]@localhost`.
    3. Save and close the file.

#### A.1.4. Removing A User

1. Make sure you have saved any files from the user's home directory that you want to keep.
2. `pseudo`
3. `userdel -r [username]`


### A.2. Data & Permissions

Datasets exist on a separate hard drive and is mounted to the path `/mnt/edlabs_data/`. The data is encrypted, but unlocked at boot time. The automatic mount is configured in `/etc/fstab`; but please consult a Systems Administrator.

#### A.2.1. Creating A New Project

If the data should be restricted to a subset of users, first see Section A.1.1. to create a new group, and then add selected users to that group. Otherwise, just execute the steps below. If permissions ever need to be fixed or "refreshed" on a dataset, execute steps [A.2.1.2.](#a212-load-the-data) and [A.2.1.3.](#a213-set-folder-permissions)

##### A.2.1.1. Create the Directory

Create the directory, substituting [dataset] for a name of your choice (keep to alphabetical characters and replace spaces with underscores).

`sudo mkdir /mnt/edlabs_data/[dataset]`

##### A.2.1.2. Load the Data

Copy the data into the new directory (note the directory slashes):  
`rsync –rv /path/to/data/ /mnt/edlabs_data/[dataset]/`

##### A.2.1.3. Set Folder Permissions

Before copying, set ownership on the directory. Usually the group will be edlabs, but if you created a new group for the dataset, use that group instead.
 
1. `sudo chown -R root:[group] /mnt/edlabs_data/[dataset]`

Set default permissions on the directory. This gives read/write/execute access to the owner and group, and denies access to any other account.

2. `sudo chmod -R 2770 /mnt/edlabs_data/[dataset]`

If the project directory _should_ be available to users outside the group assigned to the project, use `2777`.

####  A.2.2. Removing A Project

A dataset should be securely erased (as root) when it’s no longer needed. (This may take some time depending on the amount of data.)

1. `pseudo`
2. `srm -rf /mnt/edlabs_data/[dataset]`
3. `exit`

To remove users from a group, see Section A.1.1.4. To remove users from the system, see Section [A.1.4.](#a14-removing-a-user)


### A.3. Security

####  A.3.1. Encryption Passphrases

Both the operating system and data are encrypted by default, each with their own passphrase. The disks are unlocked at boot and remain unlocked until shutdown or reboot. The passphrases can be changed independent of each other at any time, by an administrator. You will need to specify the encrypted partition to change the passphrase; the defaults are:

* Operating System (e.g. applications, user home directories): `/dev/sda2`
* Data: `/dev/sdb1`

##### A.3.1.1. Confirming Partitions

Confirm (if desired) the encrypted partitions, using one or both of the following methods:

* `sudo blkid -p /dev/[partition] | grep -q "LUKS" && echo "encrypted device"`
* `sudo cryptsetup -v isLuks /dev/[partition]`

##### A.3.1.2. Add The Passphrase

`sudo cryptsetup luksAddKey /dev/[partition]`

##### A.3.1.3. Remove A Passphrase

`sudo cryptsetup luksDeleteKey /dev/[partition]`

####  A.3.2. System Passwords

Besides user accounts, there are several instances where passwords are used to secure aspects of the system.

##### A.3.2.1. root

The built-in administrator account. The system is configured so that the root account is inaccessible at the login screen or console. For purposes of managing the system, it’s available for elevating privileges within a user sessions.

##### A.3.2.2. BIOS

Several hardware settings are configured in BIOS, such as disabling network cards. A password helps prevent accidental or malicious changes. Although the BIOS can be reset from the system mainboard, the physical lock on the workstations prevent tampering.

##### A.3.2.3. GRUB

GRUB is the Linux bootloader. It sets parameters for what and how the operating system should boot. For example, booting into a special mode called "single user" or "rescue mode" allows the root password to be reset. Password protecting GRUB prevents accidental or malicious changes to the boot parameters.

##### A.3.2.4. LUKS

The LUKS passphrases are for disk encryption. There are two separate passphrases, one for the operating system, and another for the data. See Section [A.3.1.](#a31-encryption-passphrases) for more information.

#### A.3.3. Firewall

The firewall is configured to block all traffic, except local web applications (i.e. RStudio). If a new application requires adding firewall rules, please contact HMDC.

#### A.3.4. Intrusion Detection

The AIDE utility scans the system for changes. It can report any differences between a baseline and a separate independent scan. This allows administrators to flag suspicious changes for review. Some system changes, like adding new users for example, will be reported by AIDE, but should not be flagged as it is a user-initiated change. For reviewing AIDE logs, see Section [A.4.3.](#a43-aide-logs)

##### A.3.4.1. Resetting AIDE Baseline

An original baseline is created at the time the systems are delivered to EdLabs, but may need to be updated periodically as more work is done. If the AIDE logs become too long, create a new baseline as follows:

1. Become root (use the custom alias): `pseudo`
2. Initialize a new AIDE database (can take a few minutes): `aide --init`
3. Copy the new database into place: `cp -p /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz`


### A.4. System Logs

#### A.4.1. Audit

The Audit daemon is configured to log all root commands and access to data. The audit log is found at `/var/log/audit/auditd.log`.

* To display all root commands: `ausearch -ua 0`
* To display access to data: `ausearch -f /etc/edlabs_data`

#### A.4.2. Archived Logs

All logs are archived to `/var/log/logs_archive/` within monthly directories. They are available at any point for review.

#### A.4.3. AIDE Logs

The AIDE log is found at `/var/log/aide/aide.log`. It is created nightly by a cronjob that runs `aide --check` on the system. The resulting log is then emailed to the administrators. Interacting directly with the log file is usually not necessary.


### A.5. Securely Erasing Hard Drives

To securely erase individual files, see Section [B.3.1.](#b31-securely-erasing-files) For disk erasure, please contact a system administrator (currently HMDC). The nwipe program installed on the system should only be used by a knowledgeable IT staff member. EdLabs staff should not attempt to erase a disk without consulting HUIT or HMDC.

In the event a disk needs to be wiped of data, it should be removed from the RAID group and re-mounted as a stand-alone drive, as to not destroy any other critical data. If possible, the disk should be transported to HUIT or HMDC for proper wiping and destruction.


### B.1. Account Management

#### B.1.1. Account Expiration

All accounts expire after one year, and require an administrator to extend by another year. You will receive mail notices starting one month before expiration. You can check mail via the command line with the mail package, or setup Thunderbird; see Section [B.1.2.](#b12-reading-mail)

#### B.1.2. Reading Mail

All accounts receive mail; regular users, for example, will be notified of pending account expiration. Administrators receive logs and account expiration notices as well. Users can read mail through the Thunderbird application.

##### B.1.2.1. Configuring Thunderbird

1. Applications &#8594; Internet &#8594; Thunderbird
2. Select "I think I’ll configure my account later."
3. Menu &#8594; Preferences &#8594; Account Settings
4. Account Actions &#8594; Add Other Account
5. Choose "Unix Mailspool (Movemail)"
6. Identity information
    1. Enter your first and last name under "Your Name"
    2. Replace "(none)" with "localhost" so it reads `[username]@localhost`
7. Outgoing Server Information
    1. Outgoing Server: `localhost`
    2. Outgoing User Name: `[username]`
8. Account Name can be renamed if desired

Mail is usually delivered by the system, but it is also possible to write mail person to person by sending to `[username]@localhost`.


### B.2. Applications

#### B.2.1. Running Applications

Custom installed applications and packages are grouped together and can be found under Applications &#8594; EdLabs in the upper-left menu. Other applications are organized by function.

#### B.2.2. Software Collections

Software Collections is a special repository of newer packages than what ship standard with CentOS 7. There are several installed; list them with `scl -l`. See https://www.softwarecollections.org/en/scls/ for reference. Activate an environment with `scl enable [collection] bash`. To exit, just type `exit`.

#### B.2.3. Using Python

To open the default 2.7 interpreter, use the EdLabs menu and select "Python 2.7", or from a command line type `python`. Python 3.4 is also available. Using the newer version of Python requires activating the 3.4 environment (`scl enable python34 bash`). (If you select "Python 3.4" from the EdLabs menu, a terminal running the python34 environment will open, it’s simply a shortcut.) Then run python like you normally would.

You can check the version of Python you’re using with `python --version`.

##### B.2.3.1. Installing Python Modules

Any modules not already installed need to be downloaded from an internet connected machine and copied over to the level 5 workstation. See https://github.com/IQSS/Basket-Helper.

Once copied over, install with `pip install --user [module_name]`.


### B.3. File Management

#### B.3.1. Securely Erasing Files

Use the srm package to securely erase a file. Deleting a file with rm or through the file manager will not properly overwrite the data. The full command is `srm [file]`. For multiple files, separate the filenames with spaces. For directories with sub-directories and/or files, you can secure delete it all at once with `srm -R [directory]`.

**Note:** `srm` is an alias for `srm -D` to ensure a 7-pass overwrite. If file permissions do not allow for deletion, see Section [B.3.2.1.](#b321-fixing-permissions)

####  B.3.2. File Permissions

When copying and creating files under `/mnt/edlabs_data`, files (and folders) will automatically inherit the group ownership "edlabs", which (usually) all users will belong to. This allows for collaborative work.

Note that if files are cut/pasted, or moved into `edlabs_data`, the original ownership and permissions will persist, which may preclude access by other team members. It is recommended to copy data only (or use rsync without the ‘archive’ or ‘retain ownership/permissions’ flags, see Section [A.2.1.3.](#a213-set-folder-permissions)) when moving it onto the system.

##### B.3.2.1. Fixing Permissions

An administrator is required to fix permissions. For folders, run `sudo chmod -R 2770 [folder]` or `sudo chmod 770 [file]` for files.


### B.4. Flash Drives

#### B.4.1. General Usage

CentOS 7 has a new default location for mounting flash drives: `/run/media/[username]/[drive]`. Upon inserting a flash drive, a popup will appear at the bottom of the screen to indicate that the system has automatically mounted the drive. A new icon will also appear on the desktop. Right-click to eject and/or remove the drive.

#### B.4.2. Using an IronKey

##### B.4.2.1. Manual Instructions

Once the IronKey has been inserted, an unencrypted partition will automatically mount. This partition contains utilities for managing the IronKey, such as unlocking the encrypted data partition. Once unlocked, the data partition will also automatically mount.

**Old style IronKey:**

* Unlock: `/run/media/[username]/IronKey/linux/ironkey`
* Lock: `/run/media/[username]/IronKey/linux/ironkey --lock`

**New style IronKey:**

* Unlock: `run/media/[username]/IRONKEY/linux/ironkey.exe`
* Lock: `/run/media/[username]/IRONKEY/linux/ironkey.exe --lock`

After locking the IronKey, you will still need to unmount the drive (both data and utility partitions) by clicking the eject button, or right-clicking and selecting unmount/eject/remove (depending on partition).

##### B.4.2.2. Using the Custom Script

A system script automates the entire process, no matter which version IronKey you are using. From a terminal:

`sudo ironkey`

Use the menu to:

* Option 1: unlock the IronKey
* Option 5: re-lock the IronKey
* Option 9: exit the script
