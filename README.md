# rsyncsnap - Rsync Incremental Backups

Bash script that uses rsync to create multiple incremental hard link snapshots of single or multiple directories to local or remote ssh destination.

----------------------------------------------------------------------------------

## Features

- Can perform push or pull backups.  Does push on default.  Use --pull and --sudo for pull backups.
- Can create and store backup snapshots on local machine or remote server using ssh.
- Snapshots are Samba `vfs_shadow_copy2` compatible with Windows Previous Versions.
- Uses updated soft link to most current backup within backup directory to keep track of previous snapshot.
- Using single directory as source will backup single directory.
- Using include file as source will backup directories listed in file.
- The use of .includes file is for adding single or multiple sources to backup.  Can use any name for the file and extension.
- The use of .excludes file is for excluding directories and files patterns from backup.  Can use any name for the file and extension.
- Destination directory (snapshot archive) will be named the same as what the include filename is.  (mybackup.include = /mnt/ext-backup/rsyncsnap/mybackup@2020.03.18-15.01.25)  Can have multiple backup archives with their own custom name in same destination.  Can use --backup-name or --name to use that name instead of what file/source is named.

----------------------------------------------------------------------------------

- Recommended to name backup destination directory "rsyncsnap" to help keep track and knowing how your backup snapshots were made.  (/mnt/ext-backup/rsyncsnap)
- Ranger (ranger.github.io) makes it easier to browse snapshots in destination when using shell when have a lot of snapshots in destination.  Or use any file manager you like be is text or GUI.

----------------------------------------------------------------------------------

## Warnings
- Do not delete the soft link to rsyncsnap-current directory!  The soft link to rsyncsnap-current directory is most important and how rsyncsnap keeps track of previous backup.
- Be careful with backup snapshot number.  If snapshot amount is set to a low number like 1 then all snapshots will be removed.
- Changing permissions on destination directories after a snapshot backup was performed can cause issues during next snapshot run since rsync is set to preserve owner and group permissions.  It is best to not change anything at the destination.
- Be careful when using --snap-amount and --snap-days together as --snap-days may remove more snapshots than planned if not used properly.

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
vi /root/rsyncsnap.exclude
```

#### Example directory listing of backups: <%Y-%m-%d_%H.%M.%S>
```
ls
/mnt/backups/rsyncsnap/.rsyncsnap-current
/mnt/backups/rsyncsnap/rsyncsnap@2020-03-15_14.58.11
/mnt/backups/rsyncsnap/rsyncsnap@2020-03-16_14.58.40
/mnt/backups/rsyncsnap/rsyncsnap@2020-03-18_14.59.07
/mnt/backups/rsyncsnap/rsyncsnap@2020-03-18_14.59.10
/mnt/backups/rsyncsnap/rsyncsnap@2020-03-18_15.01.25
``` 

----------------------------------------------------------------------------------

## Usage

```
-------------------------------------------------------------------------------
rsyncsnap

Uses rsync to create incremental hard link snapshots of directories.

https://github.com/nicedreams/rsyncsnap
-------------------------------------------------------------------------------
- Performs push backup from local to remote server by default.
- To perform pull backup from remote server to local use
  --pull and --sudo options.
--------------------------------------------------------------------------------
USAGE:

  rsyncsnap  <source>  <destination>  <options>

  # Using include file (rsyncsnap.include)
  rsyncsnap /root/rsyncsnap.include /mnt/backups \\
            --exclude /root/rsyncsnap.exclude \\
            --snap-amount 30 \\
            --snap-days 45 \\
            --logfile /var/log/rsyncsnap.log \\
            --mail user@domain \\
            --syslog

  # Using single source directory (/home/user)
  rsyncsnap /home/user /mnt/backups \\
            --snap-days 60 \\
            --mail user@domain

  # Using include file to SSH destination (user@server:/)
  rsyncsnap /root/rsyncsnap.include user@server:/home/user/backups \\
            --exclude /root/rsyncsnap.exclude \\
            --snap-amount 30 \\
            --logfile /var/log/rsyncsnap.log
            
  # Performing pull backup from remote server to local directory          
  rsyncsnap user@server /srv/backups/
            --pull \":/etc/ :/home/ :/usr/local/\" \\
            --sudo rsync \\
            --exclude /home/user/rsyncsnap.exclude \\
            --snap-amount 14 \\
            --logfile /var/log/rsyncsnap-pull.log

  <source>        Full path and filename of directory or rsyncsnap.include file
  <destination>   Where to store backups
  <options>       Listed below
--------------------------------------------------------------------------------
- Using single directory as source will backup single directory.
- Using include file as source will backup directories listed in file.
- Can use multiple source directories within \"quotes\"
  \"/home/user/ /etc/ /usr/local/bin/\"
--------------------------------------------------------------------------------
OPTIONS:

  --snap-amount | Amount of snapshots to keep.
                  --snap-amount 30                      (Keep 30 total snapshots)

  --snap-days     How many days to keep snapshots.
                  --snap-days 45                      (Keep 45 days of snapshots)
                  **Not recommended when using SSH for destination**

  --exclude |     Path of exclude file
                  --exclude /home/user/rsyncsnap.exclude
-------------------------------------
  --logfile       Create/Append log to single file

  --mail |        Send mail using mail command
  --email         --mail user@domain.com  OR  --mail root
                  
  --syslog        Send status to syslog using logger
-------------------------------------
  --rsync-options Default: ${rsync_options[*]}                                                                                                                                                                
                  Use custom rsync options instead of built-in default.
                  **Must Use \"quotes\"
                  --rsync-options \"-axHAWXS --progress\"

  --numeric-ids   Use --numeric-ids option when using rsync to use numeric
                  user/group rather than uid/gid
-------------------------------------
  --pull          Perform pull backup from remote server to local destination
                  --pull <remote directories>
                  --pull \":/etc/ :/home/ :/usr/local/\"

  --sudo          Use sudo on remote server to elevate permissions while rsync
                  --rsync-path=\"sudo user\"
-------------------------------------
  --ssh-options   Include options for SSH
                  --ssh-options \"-p 22 -i /home/user/.ssh/keyfile\"

  --ssh           Force using SSH as destination. Use if having issues.
-------------------------------------
  --dryrun        Perform rsync version backup using --dry-run to test backup.
                  No actual backup will be performed.
                  **Use without -logfile --syslog --email options.**

  --debug         Use set-x bash option when running script.

  -h | --help     Print this usage information.
--------------------------------------------------------------------------------
NAME OF BACKUP:
  - Name of backup is set using first part of include filename or the source
    directory name if doing single directory backup.
  - Can name include file and extension anything and extension is not required.
  - Recommend using rsyncsnap as filename when using includes file to keep track
    of how backup was performed.

SSH:
  - Use --ssh-options to add regular SSH arguments to command
    --ssh-options \"-p 22 -i /home/user/.ssh/key\"
  - Script will preserve hardlinks to remote system as long as using filesystem
    that supports hardlinks like ext4, btrfs, etc.
  - Recommended to use ssh key authentication for cron jobs using
    ~/.ssh/config file.
  - Need to be able to login to remote server as root when backing up /etc or
    other files a normal user does not have access to or will get errors.
  - Recommend modify remote server /etc/sshd_config with:
    PermitRootLogin without-password

SNAPSHOTS:
  - Can keep snapshots by amount and/or by days using either one or both of the
    below options.
  - Excessive amount or older days of snapshots will be deleted.
  - If no options are used then no snapshots will be deleted.
  - Deleting by days is based on folder timestamps and system time.
  - Recommend not using --snap-days when not sure if remote system (SSH) time
    or modification times are accurate.  Use --limit-amount instead.

WARNINGS:
  - Changing permissions on destination directories after a snapshot backup was
    performed can cause issues during next snapshot run since rsync is set
    to preserve owner and group permissions.
  - Use caution to what user is performing backup. Changing user after inital
    snapshots were created can cause issues.
  - Using --numeric-ids can help prevent the issues above.
  - Be careful when using --snap-amount and --snap-days together.
    Recommended to use one, but can use both if set properly.

--------------------------------------------------------------------------------
RESTORE:
  To restore files from backup, manually copy files from backup date/time
  destination directory to original or alternate location.

--------------------------------------------------------------------------------

"
```

----------------------------------------------------------------------------------

## Examples (Modify to work with your setup)

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
ExecStart=/usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snap-days 30 --logfile /var/log/rsyncsnap.log --mail root

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
 00 02 * * * /usr/local/bin/rsyncsnap /home/user /mnt/ext-backups/rsyncsnap --snap-amount 30 --mail user
 00 03 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include /mnt/ext-backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snap-days 30 --logfile /var/log/rsyncsnap.log --mail root
 00 04 * * * /usr/local/bin/rsyncsnap /root/rsyncsnap.include ssh-server:/home/backups/rsyncsnap --exclude /root/rsyncsnap.exclude --snap-amount 15 --logfile /var/log/rsyncsnap.log --mail user@domain
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

To restore files or directories you will find all backup versions in the destination directory depending on the date/times of the snapshots and find the files you need to restore and then manually copying where they need to be restored to.  Using a program called Ranger helps to make it easier to navigate through all the snapshot directories and can preview the files from there too.  Can do via linux shell, graphic file manager within linux server or Windows desktop computer using VSS "Windows Previous Versions".

----------------------------------------------------------------------------------

## Checking and Verifying

Your data is important and you can verify if things are working properly.  If you want to verify that snapshot hardlinks are working and not copies of the file, you can compare the inodes of files within multiple directories in the destination to see if they are the same.  If a file that wasn't modified shares the same inode with another file not modified within the backup then it is a hardlink and written to the file system only once and is working properly.  Directories will not share the same inode like files do, and is working fine so only check files.

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

Show inode (hardlinks) for /mnt/backupdrive/rsyncsnap-2020-08-23_19.19.30/username/.bashrc in /mnt/backupdrive/rsyncsnap-2020-08-23_19.19.30/username/
```
find /mnt/backupdrive/rsyncsnap-2020-08-23_19.19.30/username/ -name .bashrc -type f -ls
```
