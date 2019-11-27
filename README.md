# rsyncsnap - Rsync Incremental Backups

Bash script that uses rsync to create multiple incremental backup snapshots of one or multiple sources to a destination using the previous backup.  Works with local drives and with ssh remote systems.  Be sure to use a filesystem that supporting hardlinks like ext4.

#### Example directory listing of backups: <GMT-%Y.%m.%d-%H.%M.%S>
```
ls
/mnt/backups/rsyncsnap/rsyncsnap-current
/mnt/backups/rsyncsnap/rsyncsnap@GMT-2019.07.15-14.58.11
/mnt/backups/rsyncsnap/rsyncsnap@GMT-2019.07.16-14.58.40
/mnt/backups/rsyncsnap/rsyncsnap@GMT-2019.07.18-14.59.07
/mnt/backups/rsyncsnap/rsyncsnap@GMT-2019.07.18-14.59.10
/mnt/backups/rsyncsnap/rsyncsnap@GMT-2019.07.18-15.01.25
``` 

----------------------------------------------------------------------------------

## Features

Uses rsync to create incremental snapshots with hard links to a destination directory.

- Can create and store backup snapshots on local machine or remote server using ssh.
- Snapshots are Samba `vfs_shadow_copy2` compatible with Windows Previous Versions.
- A soft link will be updated to most current backup within backup directory.
- The use of .includes file is for adding single or multiple sources to backup.  You can use any name for the file and extension.
- The use of .excludes file is for excluding directories and files patterns from backup.  You can use any name for the file and extension.
- Destination directory (snapshot archive) will be named the same as what the include filename is.  (mybackup.include = /mnt/ext-backup/rsyncsnap/mybackup@GMT-2019.07.18-15.01.25)  Can have multiple backup archives with their own custom name.

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

https://github.com/nicedreams/rsyncsnap

Uses rsync to create incremental snapshots with hard links.
Only changed files use disk space.

Add directories to backup in rsyncsnap.include file (mandatory)
Add directories to exclude in rsyncsnap.exclude file (optional)
Use include/exclude examples from git repo README if needed.

Name of backup is set using first part of filename.  Can name include
file anything you want and extension is not required.

USAGE:
  rsyncsnap <include_file> <destination> <snapshots> <options>

  rsyncsnap /root/rsyncsnap.include /mnt/backups/rsyncsnap 15 \\
            --exclude /root/rsyncsnap.exclude \\
            --logfile /var/log/rsyncsnap.log \\
            --email user@domain \\
            --syslog

  rsyncsnap /root/rsyncsnap.include \\
            user@domain:/home/user/backups/rsyncsnap 15 \\
            --exclude /root/rsyncsnap.exclude \\
            --logfile /var/log/rsyncsnap-remote.log

  <include_file>  Full path and filename of rsyncsnap.include file
  <destination>   Where to store backups: /mnt/backups/rsyncsnap
  <snapshots>     Amount of snapshots to keep: 15

CAUTION:        If snapshot amount is set to 1, all snapshots will be
                removed and be left with a single backup.

SSH NOTICE:     - Script will preserve hardlinks to remote system as
                  long as using filesystem that supports hardlinks
                  like ext4.
                - Recommended to use ssh key authentication for cron
                  jobs using ~/.ssh/config file.
                - Need to be able to login to remote server as root
                  when backing up /etc and other system files.
                  Recommend modify remote server /etc/sshd_config with:
                    PermitRootLogin without-password
                  to allow root login only with ssh keys.

OPTIONS:
  -e | --exclude  Path of exclude file: /home/user/rsyncsnap.exclude

  -l | --logfile  Path of logfile: /var/log/rsyncsnap.log

  --syslog        Send SUCCESS/ERROR message to syslog using logger

  --email         Send email (mail command)
                  --email user@domain.com  OR  --email root

  --datetime      Default: GMT-%Y.%m.%d-%H.%M.%S
                  Change date/time formatting of backup directory
                  Uses linux date command formatting
                  --datetime %m.%d.%Y-%H.%M

  --test          Perform rsync version backup using --dry-run to test
                  backup.  No actual backup will be performed.
                  Use without -l, --syslog or --email options.

Single Run Options:
  -du | --size    Get accurate size of entire backup destination (du)
                  --size /mnt/backups/rsyncsnap/

  --hardlinks     Check inode hardlink info of file in backup
                  --hardlinks <backup_destination> <filename_to_check>
                  --hardlinks /mnt/backups/rsyncsnap filename.ext

  --help          Print this usage information

RESTORE:
  To restore files from backup, manually copy files from backup
  date/time destination directory to original or alternate location.
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
  vfs objects = shadow_copy2
  shadow:snapdir = /mnt/ext-backup/rsyncsnap
  shadow:basedir = /srv/files/share
  shadow:format = rsyncsnap@GMT-%Y.%m.%d-%H.%M.%S
  shadow:fixinodes = yes
  shadow:localtime = no
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
