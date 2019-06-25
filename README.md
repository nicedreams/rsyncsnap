# rsyncsnap - Rsync Incremental Backups

Bash script that uses rsync to create multiple incremental backup snapshots of one or multiple sources to a destination using the previous backup.

Why create this when rsnapshot exists?  Started as a bash script for version control of a linux samba server in a Windows client environment.  Needed a way to work with "Windows Previous Versions" from Windows desktops and samba vss with it's date formatting of folders.  Also liked the fact that all snapshots are date/time stamped for easier restore where rsnapshot uses alpha.0 alpha.1 alpha.2 and so on.  And it's bash so easy for me to modify.  As the script grew it turned into a bash alternative to rsnapshot pretty much.  Since rsyncsnap has been working fine for my production setups I decided to share as an alternative.

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

Warning: Do not delete the soft link to rsyncsnap-current directory!  The soft link to rsyncsnap-current directory is most important and how rsyncsnap keeps track of previous backup.

----------------------------------------------------------------------------------

## Features

Uses rsync to create incremental snapshots with hard links to a destination directory.

- Snapshots are Samba `vfs_shadow_copy2` compatible with Windows Previous Versions.
- A soft link will be updated to most current backup within backup directory.
- The use of .includes file is for adding single or multiple sources to backup.  You can use any name for the file and extension.  Just something easy to keep track of.
- The use of .excludes file is for excluding directories and files patterns from backup.  You can use any name for the file and extension.  Just something easy to keep track of.
- Destination directory (snapshot archive) will be named the same as what the include filename is.  (mybackup.include = /mnt/ext-backup/rsyncsnap/mybackup@GMT-2019.07.18-15.01.25)  This way you can have multiple backup archives with their own custom name if needed.

----------------------------------------------------------------------------------

- Recommended to name backup destination directory "rsyncsnap" to help keep track and knowing how your backup snapshots were made.  (/mnt/ext-backup/rsyncsnap)
- Ranger (ranger.github.io) makes it easier to browse snapshots in destination when using shell when have a lot of snapshots in destination.

----------------------------------------------------------------------------------

## Installation

Copy rsyncsnap file to /usr/local/bin or any other directory and give execute permissions.

```
cp rsyncsnap /usr/local/bin/rsyncsnap
chmod +x /usr/local/bin/rsyncsnap

```

Create rsyncsnap.include and rsyncsnap.exclude files.  Use the example files in this README or the to help you with your own includes and excludes.  Follows normal rsync patterns.  rsyncsnap.excludes creation is optional if not needed.

```
vi /root/rsyncsnap.include
vi /root/rsyncsnap.exclude
```

#### Set cron job to run once or multiple times a day for daily snapshots

crontab -e (Run once a day at 3:00AM)

```
#00 03 * * * /usr/local/bin/rsyncsnap <include_file> <destination> <snapshots> <options>
 00 03 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap 30 --exclude /root/rsyncsnap.exclude --logfile /var/log/rsyncsnap.log --email root
```

----------------------------------------------------------------------------------

## Usage

```
rsyncsnap

Uses rsync to create incremental snapshots with hard links.
https://github.com/nicedreams/rsyncsnap

USAGE:
  rsyncsnap <include_file> <destination> <snapshots> <options>
  rsyncsnap /root/rsyncsnap.include /mnt/backups/rsyncsnap 30 \
            --exclude /root/rsyncsnap.exclude \
            --logfile /var/log/rsyncsnap.log \
            --email user@domain \
			--chmod 750 \
			--syslog

  <include_file>  Full path and filename of rsyncsnap.include file
  <destination>   Where to store backups: /mnt/backups/rsyncsnap
  <snapshots>     Amount of snapshots to keep: 30

  Caution:        If snapshot amount is set to 1 then all snapshots will be removed.

OPTIONS:
  -e | --exclude  Path and filename of exclude file: /home/user/rsyncsnap.exclude

  -l | --logfile  Path and filename of logfile: /var/log/rsyncsnap.log

  --syslog        Send message to syslog (logger)

  --email         Send email (mail command)
                  --email user@domain.com  OR  --email root

  --chmod         Change directory permissions    (default  755)
                  --chmod 750

Single Run Options:
  -du | --size    Display accurate size of entire backup destination (du)
                  --size /mnt/backups/rsyncsnap/

  --hardlinks     Check inode hardlink info on a file in backup destination
                  --hardlinks <backup_destination> <filename_to_check>
                  --hardlinks /mnt/backups/rsyncsnap filename.ext

  --help          Print this usage information
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

List inodes (hardlinks) of files within backup to compare if working

```
#rsyncsnap --hardlink <backup_location> <filename>
 rsyncsnap --hardlink /mnt/backup/ .bashrc
```
