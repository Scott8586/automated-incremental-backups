# Incremental Backups Using Rsync

* Implemented in Bash.
* Uses rsync
* Incremental backups with hardlinks.
* Automatic rotations and log-keeping.
* Lightweight, no install.
* Easy to configure, designed for use with cron.

## Usage example

The `cron` entry. (On systems that do not run continuously, using `anacron` would be more convenient.)

    # m h  dom mon dow   command
    00 05 * * * ibur /precious/files/ /backup/location/ --num-revisions 4

The directory and file structure of `/backup/location/`.

    backup_2011-06-30_05:00/
    backup_2011-07-01_05:00/
    backup_2011-07-03_05:00/
    backup_2011-07-04_05:00/
    backup_current -> backup_2011-07-03_05:00/
    backup.log

Content of the log file `backup.log`.

    2011-06-28_05:00  -  `backup_current' not found.
    2011-06-28_05:00  -  New backup created.
    -----------------------------------------------------------
    2011-06-29_05:00  -  Up to date. No backup created.
    -----------------------------------------------------------
    2011-06-30_05:00  -  Incremental backup created.
    -----------------------------------------------------------
    2011-07-01_05:00  -  Incremental backup created.
    -----------------------------------------------------------
    2011-07-02_05:00  -  Up to date. No backup created.
    -----------------------------------------------------------
    2011-07-03_05:00  -  Incremental backup created.
    -----------------------------------------------------------
    2011-07-04_05:00  -  Incremental backup created.
    2011-07-04_05:00  -  Removed 'backup_2011-06-28_05:00'
    -----------------------------------------------------------

## Doc / help

        ibur - Incremental Backups Using Rsync v. 0.03
    Automates creation and rotation of incremental backups via rsync.

    Usage: ibur [OPTION]... SOURCE DEST

    Note: This version can only do local -> local and remote -> local backups.
    Backing up to remote is easiest done by mounting remote with sshfs or similar.

    Mandatory arguments to long options are mandatory for short options too.
      -n, --num-revisions=NUM  Number of revisions to be kept. The oldest backups
                                 exceeding NUM will get deleted automatically.
                                 Default: `0=inf'.
      -d, --no-default         Do not use the pre-set arguments passed to rsync;
                                 `--archive'.
      -l, --no-logging         Disable automatic logging. Default is to log actions
                                 and errors to `DEST/PREFIX.log'.
      -p, --prefix=PREFIX      Prefix backup directories and the log-file with
                                 this. Default: `backup'. This also affects what
                                 files are used for include and exclude -- see
                                 `Additional Information'.
      -e, --rsh=COMMAND        Only for use when DEST is remote! Use this command to
                                 log in to the remote shell. This is used for
                                 maintaining backups (rotations etc.) This will NOT
                                 get passed to rsync, use `--' for additional rsync
                                 options. Also.. this is not yet implemented :)
                                 mount remote with sshfs or similar insted.
      -- ARG1 ARG2 ...         Separator for arguments supplied to rsync. All
                                 arguments following this option are directly passed
                                 to rsync. Useful for setting `--rsh' option to
                                 rsync when SRC is remote.

    Additional Information:
      - If no change at all is detected between current data and last made backup
        (`DEST/PREFIX_current'), then no new backup is created.
      - rsync options
          `--exclude-from=DEST/PREFIX.exclude' and/or
          `--include-from=DEST/PREFIX.include'
        are automatically used if respective file exists.
      - Directory structure in DEST: one directory per backup;
        `PREFIX_YYYY-MM-DD_HH:MM' and symlink `PREFIX_current' linking to latest
        backup.
      - Specifying how far back in time backups are to be kept cannot be done
        directly, only the number of backups can be specified. The frequency of
        backing up is specified when running cron.

    Example Usage:
      `ibur /precious/files/ /backup/location/ -n 8 -- -x -y'

## ToDo

* Add support for specifying how far in back in time backups should be stored.
  Currently it is only possible to specify the number of backups to be kept.
* Add support for local -> remote backups, this is rather problematic, rsnapshot
  struggles witht he same issue. Mount remote with sshfs or similar instead.
