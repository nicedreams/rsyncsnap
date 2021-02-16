# rsyncsnap - Rsync Incremental Backups

Bash script that uses rsync to create incremental hard link snapshots of directories to local or remote destination.

----------------------------------------------------------------------------------

## Features

- Creates date/time stamped folders every backup.
- Can perform push or pull backups.
- Can create and store backup snapshots on local machine or remote server using ssh.
- Snapshots are Samba `vfs_shadow_copy2` compatible with Microsoft Windows Previous Versions.
- Uses updated relative soft link to most current backup within backup directory to keep track of previous snapshot.
- Using single directory as source will backup single directory.
- Using include file as source will backup multiple directories listed in file.
- The use of .includes file is for adding single or multiple sources to backup.  Can use any name and file and extension.
- The use of .excludes file is for excluding directories and files patterns from backup.  Can use any name and file and extension.

----------------------------------------------------------------------------------

## Warnings
- Do not delete the soft link to rsyncsnap-current directory!  The soft link to rsyncsnap-current directory is most important and how rsyncsnap keeps track of previous backup when creating hardlinks.
- Be careful with backup snapshot number.  If snapshot amount is set to a low number like 1 then all snapshots will be removed.
- Making any modifications to permissions, directories or files on destination directories after a snapshot backup was performed can cause issues during next snapshot run.  It is best to not change anything at the destination and consider it **read-only**.
- Be careful when using `--snapshots` and `--days` together, as `--days` may remove more snapshots than planned if not planned properly.  `--days` option will not work with remote ssh destinations.

## Installation

Copy rsyncsnap file to `/usr/local/bin` or any other directory and give execute permissions.
```
cp rsyncsnap /usr/local/bin/rsyncsnap
chmod +x /usr/local/bin/rsyncsnap
```

Copy `rsyncsnap.include` and `rsyncsnap.exclude` files to `/root` or another directory of your choosing or create your own includes file.

Create `rsyncsnap.include` and `rsyncsnap.exclude files`.  Use the example files to help you with your own includes and excludes.  Follows normal rsync patterns.  Excludes creation is optional if not needed.
```
vi /root/rsyncsnap.include
vi /root/rsyncsnap.exclude
```

#### Example directory listing of backups: <Year-Month-Day_Hour.Minute.Second>
```
ls
/mnt/backups/rsyncsnap/rsyncsnap-current
/mnt/backups/rsyncsnap/rsyncsnap@2021-01-15_14.58.11
/mnt/backups/rsyncsnap/rsyncsnap@2021-01-16_14.58.40
/mnt/backups/rsyncsnap/rsyncsnap@2021-01-18_14.59.07
/mnt/backups/rsyncsnap/rsyncsnap@2021-01-18_14.59.10
/mnt/backups/rsyncsnap/rsyncsnap@2021-01-18_15.01.25
``` 

----------------------------------------------------------------------------------

## Usage

```
rsyncsnap
Uses rsync to create incremental hard link snapshots of directories.

USAGE:

  rsyncsnap  <source>  <destination>  [options]

  <source>        Full path and filename of directory or include file
  <destination>   Where to store backups
  <options>       Listed below
--------------------------------------------------------------------------------
OPTIONS:

-s --snapshots   Amount of snapshots to keep:
                 --snapshots 30  (Keep 30 total snapshots)

-d --days        How many days to keep snapshots:
                 --days 45  (Keep 45 days of snapshots)
                 *Does not work when using remote/ssh as destination

-e --exclude     Path of exclude file: --exclude /home/user/rsyncsnap.exclude
-------------------------------------
-l --logfile     Directory and filename location to create/append log file:
                 --logfile /home/user/rsyncsnap.log

-m --mail        Send email using mail command:
--email          --mail user@domain.com // --mail root
                  
--syslog         Send status to syslog using logger
-------------------------------------
-ro              Default: ${rsync_options[*]}
--rsync-options  Use custom rsync options instead of built-in default.
                 **Must Use "quotes"
                 --rsync-options "-avhxRHAWXS --numeric-ids" (Copy all permissions)
                 --rsync-options "-rptgoDvhxSR --numeric-ids" (No copy symlinks)
-------------------------------------
--pull           Perform pull backup from remote server to local destination
                 Directories must have : <colon> before their names
                 Must Use "quote string of arguments"
                 --pull "<remote directories>"
                 --pull ":/etc/ :/home/ :/usr/local/"

--pull-sudo      Use sudo option with ssh:
                 **Must Use "quotes"
                 --rsync-path="sudo user"
-------------------------------------
-v | --verbose   Display more verbose output like snapshot amount information
--dryrun         Perform rsync using --dry-run to test backup.
--debug          Use set-x bash option when running script.
-V | --version   Version information
-h | --help      Print this usage information.
--------------------------------------------------------------------------------
EXAMPLES:

  rsyncsnap /root/rsyncsnap.include /mnt/backups \
            --exclude /root/rsyncsnap.exclude \
            --snapshots 30 \
            --days 45 \
            --logfile /var/log/rsyncsnap.log \
            --mail user@domain \
            --syslog

  rsyncsnap /home/user /mnt/backups
  rsyncsnap /root/rsyncsnap.include user@server:/home/user/backups

PULL BACKUP FROM REMOTE SERVER USING SSH:

  rsyncsnap <remote_user@server_to_backup> <local_destination_to_store_backups>
            --pull "<:directories :on :remote: :server>"
  
  rsyncsnap user@server /srv/backups/
            --pull ":/etc/ :/home/ :/usr/local/" \
            --pull-sudo user
--------------------------------------------------------------------------------
- Using single directory as source will backup single directory.
- Using include file as source will backup directories listed in file.
- Can use multiple source directories within "quotes"
  "/home/user /etc /usr/local/bin"
- Symlink to -current is relative so can move backup directory without issues.
--------------------------------------------------------------------------------
NAME OF BACKUP:
- Backup is named based on include filename or the source directory name
  if doing single directory backup.
--------------------------------------------------------------------------------
RESTORE:
- To restore files from backup, manually copy files from backup date/time
  in destination directory to original or alternate location.
--------------------------------------------------------------------------------
"
```

----------------------------------------------------------------------------------

## Examples (Modify to work with your setup)

#### ./rsyncsnap.include
```
/home/
/root/
/etc/
/usr/local/
/var/www/
```

#### ./rsyncsnap.exclude
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
  shadow:snapsharepath home/user
  shadow:snapdir = /mnt/ext-backup/rsyncsnap
  shadow:format = rsyncsnap@%Y-%m-%d_%H.%M.%S
  follow symlinks = yes
  wide links = yes
  allow insecure wide links = yes
  shadow:localtime = yes
```

#### Set systemd timer to run once a day for daily snapshots

/etc/systemd/system/rsyncsnap.timer

```
[Unit]
Description=Run rsyncsnap hard link backup script

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=false
 
[Install]
WantedBy=timers.target
```

/etc/systemd/system/rsyncsnap.service

```
[Unit]
Description=Run rsyncsnap hard link backup script

[Service]
Type=simple
User=username
ExecStart=/usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snapshots 30 --logfile /var/log/rsyncsnap.log --mail root

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl start rsyncsnap.timer && systemctl enable rsyncsnap.timer
systemctl status rsyncsnap.timer
systemctl list-timers

journalctl -u rsyncsnap.service  # view the logs
journalctl -f -u rsyncsnap.service  # tail the logs
```

#### Set cron job to run once a day for daily snapshots

crontab -e

```
#00 00 * * * /usr/local/bin/rsyncsnap <include_file> <destination> <snapshots> <options>
 00 02 * * * /usr/local/bin/rsyncsnap /home/user /mnt/ext-backups/rsyncsnap --days 30 --mail user
 00 03 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snapshots 30 --logfile /var/log/rsyncsnap.log --mail root
 00 04 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include ssh-server:/home/backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snapshots 15 --logfile /var/log/rsyncsnap.log --mail user@domain
```

#### Configure logrotate to compress logs once a week

Create a new file `/etc/logrotate.d/rsyncsnap` with contents below.  Change /path/and/name of log file to match yours.

```
/var/log/rsyncsnap.log {
  rotate 5
  weekly
  compress
  missingok
  notifempty
  mail root
  nocreate
}

```

----------------------------------------------------------------------------------

## Restoring Files

To restore files or directories you will find all backup versions in the destination directory based on the date/times of the snapshots. From there you find the files you need to restore and then manually copying where they need to be restored to.  Can use any program or if on Windows desktop computer using VSS "Microsoft Windows Previous Versions".

----------------------------------------------------------------------------------

## Checking and Verifying

Your data is important and you can verify if things are working properly.  To verify that snapshot hardlinks are working and not copies of the file, you compare the inodes of files within multiple snapshot directories in the destination to see if they are the same.  If a file that wasn't modified shares the same inode with another file not modified within the backup then it is a hardlink and written to the file system only once and is working properly.  Directories will not share the same inode like files do, and is working so only check files.

#### Use these options to check size and hardlinks of backup.

Get accurate total size of backup using du command

Show size of current backup contents with total:
```
du -sch /mnt/backupdrive/rsyncsnap-current/*
```

Show size differences of all backups using hard-links with total:
```
du -sch --exclude="/mnt/backupdrive/rsyncsnap-current" /mnt/backupdrive/rsyncsnap-current/*
```

Total of top current backup might be smaller from deleted files within current.  If total of all backups in bottom shows larger size, hard-links might not be working properly.

#### List inodes (hardlinks) of files within backup to compare if working.  This will find all files in path <filename> and list inodes for each.

If no file results shown, then file is not found

Show inode (hardlinks) for /mnt/backupdrive/rsyncsnap-2021-01-23_19.19.30/username/.bashrc in /mnt/backupdrive/rsyncsnap-2021-01-23_19.19.30/username/
```
find /mnt/backupdrive/rsyncsnap-2021-01-23_19.19.30/username/ -name .bashrc -type f -ls
```
