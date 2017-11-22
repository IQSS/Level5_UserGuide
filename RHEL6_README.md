# Using the Red Hat Linux Workstations

## Groups & Permissions

There are three EdLabs groups on the workstations: `edlabs_RA`, `edlabs_AdHealth`, and `edlabs_Premium`.

* Only members of the `edlabs_Premium` group have sudo power (admin rights)
* All users have read-only access to `/mnt/edlab_data`
* Members of `edlabs_RA` have full access to `/mnt/edlab_data`
* Members of `edlabs_AdHealth` have full access to `/mnt/edlab_data/Raw_data/AdHealth`
* Users with no group, or just `edlabs_RA`, do not have read or write access to AdHealth data

## Creating New Users

1. Log in as EdLabs Admin
2. Navigate to System &#8594; Administration &#8594; Users & Groups
3. Username should be first initial plus last name (e.g. John Smith = jsmith)
4. Assign a default password
5. Save &#8594; reopen user properties &#8594; force user to change password upon first log in
6. Add the user to the sudoers file for IronKey access:
    1. In a terminal, become root: `sudo su -`
    2. Open the sudoers file for editing: `EDITOR=gedit visudo`
    3. Scroll down to "## Same thing without a password" section
    4. Add the new user to the list
    5. Save and quit

## How to Use Root Access

* To issue a single action as root: `sudo <command>`
* To switch to root: `sudo su -` _This is the preferred method because the former does not require knowing the root password._

## Using IronKeys

The developers of the IronKey included a perl script which interacts with the encrypted portion of the flash drive; A customized version of the script is included on the system.

1. Plug in the IronKey flash drive to any USB port
2. Invoke the IronKey script: `sudo ironkey`
    * Option 1: Unlocks the drive; prompts for user’s passphrase; mounts the data partition
    * Option 5: Locks the drive; unmounts the data partition
    * Option 9: Exits the IronKey script

## External Drive Backup

1. Plug in the external hard drive to any USB port
2. A pop-up message states that the Windows partition cannot be mounted; ignore this
3. Execute `sudo /root/external_backup.sh`
    * rsync will begin copying data to the external drive
    * A log file is created at `/var/log/logrotate/external_backup_<date>`

## Checking system logs

Online guide: http://lowfatlinux.com/linux-read-email.html

Log summaries are sent to the root account (administrator) for review. The actual logs remain on the system indefinitely, and are archived under `/var/log/logs_archive/YYYY-MM/`.

1. Become root: `sudo su -`
2. Open mail: `mail`
3. Navigating the logs:
    * `h` = list all messages
    * `<number>` = display message number
    * Press `[Enter]` to scroll the message
    * `d<number>` = delete a specific message
    * `d<number>-<number>` = delete a range of messages; inclusive
    * `q` = quit mail
4. What to look for:
    * `pam_unix` - for multiple and/or repeated failed authentications
    * `sudo (secure-log)` - for users running sudo (root) commands
    * disk space - check percentage of space used
    * Tripwire - excessive system changes under  (safe to ignore all `/var/log/` changes)
