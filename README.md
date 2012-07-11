# Incremental Backups Using Rsync

* Implemented in Bash.
* Uses rsync
* Incremental backups with hardlinks.
* Automatic rotations and log-keeping.
* Lightweight, no install.
* Easy to configure, designed for use with cron.

## Example Usage

`crontab` (On systems that do not run continuously, using `anacron` would be more convenient.)

    # m h  dom mon dow   command
    00 05 * * * ibur "/precious/files/" "/backup/location/" --num-revisions 4

`/backup/location/`

    backup_auto_2011-06-30_05:00/
    backup_auto_2011-07-01_05:00/
    backup_auto_2011-07-03_05:00/
    backup_auto_2011-07-04_05:00/
    backup_auto_current -> backup_auto_2011-07-03_05:00/
    backup_auto.log

`backup_auto.log`

    2011-06-28_05:00  -  `backup_auto_current' not found.
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
    2011-07-04_05:00  -  Removed 'backup_auto_2011-06-28_05:00'
    -----------------------------------------------------------

## Doc / help

        ibur - Automated Rsync-based Incremental backup Script v. 0.02
    Automates creation and rotation of incremental backups via rsync.

    Usage: ibur [OPTION]... SOURCE DEST

    Note: This version can only do local -> local and remote -> local backups.
    Backing up to remote is easiest done by mounting remote with sshfs or similar.

    Mandatory arguments to long options are mandatory for short options too.
      -n, --num-revisions=NUM  Number of revisions to be kept. The oldest backups
                                  exceeding NUM will get deleted automatically.
                                  Default: `0=inf'.
          -- ARG1 ARG2 ...     Separator for arguments supplied to rsync. All
                                 arguments following this option is directly passed
                                 to rsync. Useful for setting `--rsh' option to
                                 rsync when SRC is remote.
      -d, --no-default         Do not use the pre-set arguments passed to rsync;
                                 `--archive'.
      -l, --no-logging         Disable automatic logging. Default is to log actions
                                 and errors to `DEST/PREFIX.log'.
      -p, --prefix=PREFIX      Prefix backup directories and log with PREFIX.
                                 Default: `backup_auto'. This also affects what
                                 files are used for include and exclude - see
                                 `Additional Information'.
      -e, --rsh=COMMAND        The remote shell to be used for maintaining backups
                                 (rotations etc.) by this script - when DEST is
                                 remote. This will NOT get passed to rsync, use `--'
                                 for additional rsync options.
                                 Not yet implemented, mount remote with sshfs or
                                 similar insted.

    Additional Information:
      - If no change at all is detected between current data and last backup
        (`DEST/PREFIX_current'), no new backup is created.
      - rsync options
          `--exclude-from=DEST/PREFIX.exclude' and/or
          `--include-from=DEST/PREFIX.include'
        are automatically used if respective file exists.
      - Directory structure in DEST: one directory per backup;
        `PREFIX_YYYY-MM-DD_HH:MM' and symlink `PREFIX_current' linking to latest
        backup.
      - Intended to be used as a cron-job. Specifying how far back in time backups
        are to be kept cannot be done directly; job frequency and number of
        revisions combined specifies this.

    Example Usage:
      Run as a cron job.
      `ibur "/precious/files/" "/backup/location/" -n 8 -- -x -y'

## ToDo

* Add support for local -> remote backups, this is rather problematic,
  rsnapshot struggles witht he same issue. Mount remote with sshfs or
  similar instead.
