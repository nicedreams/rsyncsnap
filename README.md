# rsyncsnap - Rsync Incremental Backups

Bash script that uses rsync to create multiple incremental backup snapshots of one or multiple sources to a destination using the previous backup.  Works with local drives and with ssh remote systems.  Be sure to use a filesystem that supporting hardlinks like ext4.

#### Example directory listing of backups: <%Y.%m.%d-%H.%M.%S>
```
ls
/mnt/backups/rsyncsnap/rsyncsnap-current
/mnt/backups/rsyncsnap/rsyncsnap@2019.07.15-14.58.11
/mnt/backups/rsyncsnap/rsyncsnap@2019.07.16-14.58.40
/mnt/backups/rsyncsnap/rsyncsnap@2019.07.18-14.59.07
/mnt/backups/rsyncsnap/rsyncsnap@2019.07.18-14.59.10
/mnt/backups/rsyncsnap/rsyncsnap@2019.07.18-15.01.25
``` 

----------------------------------------------------------------------------------

## Features

Uses rsync to create incremental snapshots with hard links to a destination directory.

- Can create and store backup snapshots on local machine or remote server using ssh.
- Snapshots are Samba `vfs_shadow_copy2` compatible with Windows Previous Versions.
- A soft link will be updated to most current backup within backup directory.
- The use of .includes file is for adding single or multiple sources to backup.  You can use any name for the file and extension.
- The use of .excludes file is for excluding directories and files patterns from backup.  You can use any name for the file and extension.
- Destination directory (snapshot archive) will be named the same as what the include filename is.  (mybackup.include = /mnt/ext-backup/rsyncsnap/mybackup@2019.07.18-15.01.25)  Can have multiple backup archives with their own custom name.

----------------------------------------------------------------------------------

- Recommended to name backup destination directory "rsyncsnap" to help keep track and knowing how your backup snapshots were made.  (/mnt/ext-backup/rsyncsnap)
- Ranger (ranger.github.io) makes it easier to browse snapshots in destination when using shell when have a lot of snapshots in destination.  Or use any file manager you like.

----------------------------------------------------------------------------------

## Warnings
- Do not delete the soft link to rsyncsnap-current directory!  The soft link to rsyncsnap-current directory is most important and how rsyncsnap keeps track of previous backup.
- Be careful with backup snapshot number.  If snapshot amount is set to a low number like 1 then all snapshots will be removed and be left with only 1 backup.

## Installation

Copy rsyncsnap file to /usr/local/bin or any other directory and give execute permissions.
```
cp rsyncsnap /usr/local/bin/rsyncsnap
chmod +x /usr/local/bin/rsyncsnap

# Copy rsyncsnap.include and rsyncsnap.exclude files to /root or directory of your choosing or create your own includes file.
```

Create rsyncsnap.include and rsyncsnap.exclude files.  Use the example files in this README or the to help you with your own includes and excludes.  Follows normal rsync patterns.  rsyncsnap.excludes creation is optional if not needed.
```
vi /root/rsyncsnap.include
vi /root/rsyncsnap.exclude (optional)
```

#### Set cron job to run once or multiple times a day for daily snapshots

crontab -e (Run backup to local once a day at 3:00AM) (Run backup to remote once a day at 4:00AM)

```
#00 00 * * * /usr/local/bin/rsyncsnap <include_file> <destination> <snapshots> <options>
 00 03 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap 30 --exclude /root/rsyncsnap.exclude --logfile /var/log/rsyncsnap.log --email root
 00 04 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include ssh-server:/home/backups/rsyncsnap 30 --exclude /root/rsyncsnap.exclude --logfile /var/log/rsyncsnap.log --email user@domain
```

----------------------------------------------------------------------------------

## Usage

```
rsyncsnap

Uses rsync to create incremental snapshots with hard links.
Only changed or new files use disk space.

https://github.com/nicedreams/rsyncsnap

Add directories to backup in rsyncsnap.include file         (mandatory)
Add directories to exclude in rsyncsnap.exclude file        (optional)
Use include/exclude examples from git repo README.
Can limit snapshots using --snap-amount and/or --snap-days options

Name of backup is set using first part of include filename. Can name include
file and extension anything you want and extension is not required.
Recommend using rsyncsnap as filename to keep track of how backup was performed
unless using multiple backup sets.
Backup name can also be changed using option --backup-name option overwriting
what the include file is named.

USAGE:
  rsyncsnap  <include_file>  <destination>  <options>

  rsyncsnap /root/rsyncsnap.include /mnt/backups \\
            --exclude /root/rsyncsnap.exclude \\
            --snap-amount 30 \\
            --snap-days 15 \\
            --logfile /var/log/rsyncsnap.log \\
            --email user@domain \\
            --syslog

  rsyncsnap /root/rsyncsnap.include \\
            user@domain:/home/user/backups \\
            --exclude /root/rsyncsnap.exclude \\
            --snap-amount 30 \\
            --logfile /var/log/rsyncsnap-remote.log

  <include_file>  Full path and filename of rsyncsnap.include file
  <destination>   Where to store backups: /mnt/backups

SSH NOTICE:     - Script will preserve hardlinks to remote system as long as
                  using filesystem that supports hardlinks like ext4, btrfs, etc.
                - Recommended to use ssh key authentication for cron jobs
                  using ~/.ssh/config file.
                - Need to be able to login to remote server as root when
                  backing up /etc and other system files or will get errors.
                - Recommend modify remote server /etc/sshd_config with
                    PermitRootLogin without-password

SNAPSHOTS:
                - Can keep snapshots by amount and/or by days using either one
                  or both of the below options.
                - Excessive amount or older days of snapshots will be deleted.
                - If no options are used then no backups will be deleted.
                - Deleting by days is based on folder timestamps and system time.
                - Recommend not using --snap-days if not sure if system time or
                  modification times.  Like on remote systems via SSH.
                  
  --snap-amount   Amount of snapshots to keep.
  -sa             --snap-amount 30           (Keep 30 total snapshots)

  --snap-days     How many days to keep snapshots.
  -sd             --snap-days 15             (Keep 15 days of snapshots)

OPTIONS:
  --exclude | -e  Path of exclude file: /home/user/rsyncsnap.exclude
  
  --backup-name   Name of backup.
                  Overwrites naming backup based on include file name.
                  --backup-name mybackup

  --logfile | -l  Path of logfile: /var/log/rsyncsnap.log

  --syslog        Send SUCCESS/ERROR message to syslog using logger

  --email         Send email (mail command)
                  --email user@domain.com  OR  --email root

  --datetime      Default: %Y.%m.%d-%H.%M.%S
  -dt             Change date/time formatting of backup directory
                  Uses linux date command formatting
                  --datetime %m.%d.%Y-%H.%M

  --test |        Perform rsync version backup using --dry-run to test backup.
  --dryrun        No actual backup will be performed.
                  Use without -logfile --syslog --email options.

  --debug         Adding --debug to end of full command will not run backup
                  and only list all variables for testing.
                  Good way to test if having issues with options.

Single Run Options:
  -du | --size    Get accurate size of entire backup destination (du)
                  --size /mnt/backups/rsyncsnap/

  --compare       Compare inode hardlink info of file in backup
                  --compare <backup_destination> <filename_to_check>
                  --compare /mnt/backups/rsyncsnap filename.ext

  -v | --version  Show rsyncsnap version information.
  
  -h | --help     Print this usage information.

RESTORE:
  To restore files from backup, manually copy files from backup date/time
  destination directory to original or alternate location.
```

----------------------------------------------------------------------------------

## Examples

#### /rsyncsnap.include (Use trailing slashes on directories)
```
/etc/
/root/
/home/
/usr/local/
/var/www/
```

#### /rsyncsnap.exclude (Use trailing slashes on directories)
```
*/.thumbnails
*/.cache
*/cache
*/Trash
*/tmp
*/.snapraid.content*
/var/www/example.net/
/.gvfs
```

#### Samba /etc/samba/smb.conf for Windows Previous Versions exposure.

```
[share]
  path = /srv/files/share
  [...]
  vfs objects = shadow_copy2 acl_xattr
  shadow:basedir = /srv/files/share
  shadow:snapdir = /mnt/ext-backup/rsyncsnap
  shadow:format = rsyncsnap@%Y.%m.%d-%H.%M.%S
  follow symlinks = yes
  wide links = yes
  allow insecure wide links = yes
  shadow:localtime = yes
```

----------------------------------------------------------------------------------

## Restoring Files

To restore files or directories you will find all backup versions in the destination directory depending on the date/times of the snapshots and find the files you need to restore and then manually copying where they need to be restored to.  Using a program called Ranger helps to make it easier to navigate through all the snapshot directories and can preview the files from there too.  Can do via linux shell, graphic file manager within linux server or Windows desktop computer using VSS "Windows Previous Versions".

----------------------------------------------------------------------------------

## Checking and Verifying

Your data is important and you can verify if things are working properly.  If you want to verify that snapshot hardlinks are working and not copies of the file, you can compare the inodes of files within multiple directories in the destination to see if they are the same.  If a file that wasn't modified shares the same inode with another file not modified within the backup then it is a hardlink and written to the file system only once and is working properly.  Directories will not share the same inode like files do, and is working fine so only check files.

#### Use these options to check size and hardlinks of backup.

Get accurate total size of backup using du command
```
#rsyncsnap --size <backup_location>
 rsyncsnap --size /mnt/backup/
```

List inodes (hardlinks) of files within backup to compare if working.  This will find all files in path <filename> and list inodes for each.
```
#rsyncsnap --hardlink <backup_location> <filename>
 rsyncsnap --hardlink /mnt/backup/ .bashrc
```
