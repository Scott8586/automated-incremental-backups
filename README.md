# Incremental Backups Using Rsync

## Installation

Since the program is just one file, you could simply download it from
[here](https://raw.github.com/c3l/ibur/master/ibur). Alternatively you
can clone this repository, this enables you to easily get future
updates.

I like to keep cloned repositories in `~/vc/` (Version Control). So I
use this program by cloning the repo like this:

    cd ~/vc/ && git clone git://github.com/c3l/ibur.git

Then put a symlink in `/usr/local/bin/`:

    ln -s ~/vc/ibur/ibur /usr/local/bin/

so that you can run the command `ibur` from any directory, just like any
other command.

In the future you keep up to date by doing

    cd ~/vc/ibur && git pull


## About

* Implemented in Bash.
* Uses rsync.
* [Incremental backups](http://en.wikipedia.org/wiki/Incremental_backup).
* Automatic rotations and log-keeping.
* Lightweight, no install.
* Easy to configure, designed for use with cron.

Usage is simple: `ibur /precious/files/ /backup/location/`, this will
create a copy of the files in `/precious/files/` inside
`/backup/location/backup_2012-08-23_17:52/`. That exact command can be
executed multiple times to create a new backup backup. A symlink is
created in`/backup/location/backup_latest` -- that points to the
most recent backup.

What makes this program really useful is that after a first backup has
been made, each successive backup is an incremental backup. This means
that no more data is copied than what is needed. Lets say we have 5
files, we run a backup. Then we modify one of those files, and create
a new file. Now we want to backup again; the modified file and the new
file gets copied into the backup location, but the 4 unmodified files
do not get copied. Copying them would create duplicate files from what
we have in the previous backup, instead a hardlink is created for each
unmodified file (and folder) to the previous backup. (Don't know what
a hardlink is? Don't worry, just think of it like the same file at two
places, change one and the "other" gets changed too.) This is so that
we won't use more space that we need to on our hard drive, and having
two copies of the same file is pointless. All this happens thanks to
`rsync`.

This program also creates a logfile in `/backup/location/backup.log`
that logs what is going on, this is useful when automatin this
program, see Usage example below.


## Usage example

The `cron` entry. (On systems that do not run continuously, using `anacron` would be more convenient.)

    # m h  dom mon dow   command
    00 05 * * * ibur /precious/files/ /backup/location/

The directory and file structure of `/backup/location/`.

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

The output of `ibur --help` is listed here.

        ibur - Incremental Backups Using Rsync v. 0.03
    Automates creation and rotation of incremental backups via rsync.

    Usage: ibur [OPTION]... SOURCE DEST

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
      `ibur /precious/files/ /backup/location/ -n 8 -- <extra commands to rsync>'


## Change log

- **v 0.1** 2012-08-23
  - Bugfix: `-n` swich now works properly.
  - Bugfix: relative paths now work as expected.
  - Changed name for symlink to latest backup from `backup_current`
    to `backup_latest`.
  - Improved documentation and readme.
  - Refactored code, much easier to work with now.
  - Moved todo and bugs etc. to the github issues page.
