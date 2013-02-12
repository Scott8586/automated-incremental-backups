# ibr

_Automated **I**ncremental **B**ackups using **R**sync_

## Installation

The program is just one file, no installation needed. You can simply
[download it](https://raw.github.com/c3l/ibr/master/ibr), or clone
this repository.

## About

* Implemented in Bash.
* Uses rsync.
* [Incremental backups](http://en.wikipedia.org/wiki/Incremental_backup).
* Automatic rotations, timestamps and log-keeping.
* Lightweight, no install.
* Easy to configure, easy to use with cron.

## Usage and functionality

Usage is simple: `ibr /precious/files/ /backup/location/` will create
a copy of the files from `/precious/files/` to
`/backup/location/backup_2013-02-12_17:52/`. The same command is used
each time you want to create a new backup backup. For convenience, a
symlink is created in`/backup/location/backup_latest` which points to
the most recent backup.

What makes this program really useful is that after a first backup has
been made, each successive backup is an incremental backup. This means
that no more data is copied than what is needed. Lets say we have 5
files, we run a backup. Then we modify one of those files, and create
a new file. Now we want to backup again; the modified file and the new
file gets copied into the backup location, but the 4 unmodified files
do not get copied. Copying the unmodified files to the new backup
would create unnecessary duplicates of the files, eating disk
space. But don't worry; *each backup is fully complete and contains
all files*.

This is achieved by creating hardlinks for each unmodified file (and
folder) to the last backed up version of that file. (Don't know what a
hardlink is? Don't worry, just think of it like the same file at two
places, change one and the "other" gets changed too.) This minimizes
use of disk space — having two copies of the same file is
pointless. All this happens thanks to `rsync`.

This program also creates a logfile in `/backup/location/backup.log`
that logs what is going on, this is useful when automating this
program, see the example below.


## Example

The `cron` entry.

    # m h  dom mon dow   command
    00 05 * * * /path/to/ibr /precious/files/ /backup/location/

If your system isn't running continuously, create a file
`/etc/cron.daily/autobackup` containing: (don't forget to `chmod +x`
the file.)

    #!/bin/sh
    /path/to/ibr /precious/files/ /backup/location/

The directory and file structure of `/backup/location/` created after
running `ibr` a couple of times:

    backup_2011-06-30_05:00/
    backup_2011-07-01_05:00/
    backup_2011-07-03_05:00/
    backup_2011-07-04_05:00/
    backup_latest -> backup_2011-07-03_05:00/
    backup.log

Content of the log file `backup.log`.

    2011-06-28_05:00  -  `backup_latest' not found.
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

The output of `ibr --help` is listed here.

        ibr - Incremental Backups Using Rsync v. 0.1.1
    Automates creation and rotation of incremental backups via rsync.

    Usage: ibr [OPTION]... SOURCE DEST

    Note: This version can only do local -> local and remote -> local backups.
    Backing up to remote is easiest done by mounting remote with sshfs or similar.

    Mandatory arguments to long options are mandatory for short options too.
      -h, --help               Show this help message.
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
      -- ARG1 ARG2 ...         Separator for arguments supplied to rsync. All
                                 arguments following this option are directly passed
                                 to rsync. Useful for setting `--rsh' option to
                                 rsync when SRC is remote.

    Additional Information:
      - If no change at all is detected between current data and last made backup
        (`DEST/PREFIX_latest'), then no new backup is created.
      - rsync options
          `--exclude-from=DEST/PREFIX.exclude' and/or
          `--include-from=DEST/PREFIX.include'
        are automatically used if respective file exists.
      - Directory structure in DEST: one directory per backup;
        `PREFIX_YYYY-MM-DD_HH:MM' and symlink `PREFIX_latest' linking to latest
        backup.
      - Specifying how far back in time backups are to be kept cannot be done
        directly, only the number of backups can be specified. The frequency of
        backing up is specified when running cron.

    Example Usage:
      `ibr /precious/files/ /backup/location/ -n 8 -- <extra commands to rsync>'


## Change log

- **v 0.1.1** 2013-02-12
  - [Semantic versioning](http://semver.org/).
  - Improved `README.md`.

- **v 0.1** 2012-08-23
  - Bugfix: `-n` swich now works properly.
  - Bugfix: relative paths now work as expected.
  - Changed name for symlink to latest backup from `backup_current`
    to `backup_latest`.
  - Improved documentation and readme.
  - Refactored code, much easier to work with now.
  - Moved todo and bugs etc. to the github issues page.
