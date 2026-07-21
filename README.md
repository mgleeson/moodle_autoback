# moodle_autoback

![moodle_autoback logo](logo_sml.png)

**Update:** Autoback now supports database backup for both MySQL/MariaDB and PostgreSQL.

`moodle_autoback` backs up a Moodle site's database, webroot files, and
`moodledata` directory from one command. It reads the required database and data
directory settings from Moodle's `config.php`.

Autoback started life as `moodle_db_backup`, which was a tool I built back in 2015, and the first that auto-configured itself from the Moodle config.php in order to simplify usage.
It started out as a Moodle datatbase backup tool only, but morphed into tool to backup Moodle instances more fully. 
Autoback is the next generation of its original development intention.

With the name change the old `moodle_db_backup` is no longer available (except in the git history), however the old `moodledbbackup.sh` and `moodlefilesbackup.sh` scripts are compatibility wrappers if anyone out there has actually been using it and built things that depend on those scripts (not that I expect anyone has, but just in case I figured it's best to be kind to those who might have :)) Going forward, any new use should use `moodle_autoback`.


## Usage

```bash
./moodle_autoback /path/to/moodle
./moodle_autoback ./moodle
./moodle_autoback --moodlepath /path/to/moodle
./moodle_autoback -m /path/to/moodle
```

By default, the script backs up:

* the Moodle database
* the Moodle webroot
* the Moodle `dataroot`

## Backup Location

If `--backup-path` is not supplied, the script looks beside the Moodle webroot
for a backup directory:

1. `backups/`
2. `backup/`

For example, if the Moodle webroot is:

```bash
/var/www/mymoodle.com.au/public_html
```

then the script checks:

```bash
/var/www/mymoodle.com.au/backups
/var/www/mymoodle.com.au/backup
```

If neither directory exists, backups are written to the current working
directory. The script refuses to run from inside the Moodle webroot and refuses
to write backup output inside the Moodle webroot.

## Options

```bash
./moodle_autoback /path/to/moodle --backup-path /path/to/backups
./moodle_autoback /path/to/moodle -b /path/to/backups
./moodle_autoback /path/to/moodle --db-only
./moodle_autoback /path/to/moodle --files-only
./moodle_autoback /path/to/moodle --data-only
./moodle_autoback /path/to/moodle --single-archive
./moodle_autoback /path/to/moodle --bundle
./moodle_autoback /path/to/moodle --dry-run
./moodle_autoback /path/to/moodle --debug
./moodle_autoback /path/to/moodle --email someone@example.com
./moodle_autoback --version
./moodle_autoback --help
```

`--single-archive` and `--bundle` create one final `.tar.gz` containing the
selected backup archives.

Multiple `--*-only` options can be combined. For example:

```bash
./moodle_autoback /path/to/moodle --db-only --data-only
```

## Preflight Checks

Before writing backups, the script checks:

* the Moodle path exists
* `config.php` is readable
* the backup path exists and is writable
* the current directory is not inside the Moodle webroot
* the backup path is not inside the Moodle webroot
* required commands are available
* database access works when a database backup is selected
* `moodledata` is readable when a data backup is selected
* selected file trees can be traversed before archive writing starts

Required commands for a full backup:

* `bash`
* `find`
* `tar`
* `gzip`

Database backups also require the client tools for the Moodle database type:

* MySQL/MariaDB (`mysqli`, `mariadb`, or `mysql`): `mysql` and `mysqldump`
* PostgreSQL (`pgsql`): `psql` and `pg_dump`

If `--email` is used, either `mutt` or `sendmail` is required. The `sendmail`
path also requires `base64` for attachments.

## Permissions

The user running the backup must be able to read the selected Moodle webroot and
`moodledata` trees. If Moodle session files or uploaded files are owned by the
web server user, run the backup with suitable permissions, such as from root's
cron or with `sudo`.

## Config Parsing

If PHP CLI is available, the script evaluates the Moodle `config.php` values
before Moodle's `lib/setup.php` is loaded. This supports common programmatic
config values such as paths built from `__DIR__` or environment variables.

If PHP CLI is not available, the script falls back to a Bash parser for simple
single-quoted or double-quoted `$CFG->key = "value";` assignments.

For database connections, `$CFG->dboptions['dbport']` is used when set.

## Output Files

The database backup is written as:

```bash
SITE_DB_DBNAME_backup_YYYY-MM-DD--HHMMSS.sql.gz
```

The webroot backup is written as:

```bash
SITE_files_backup_YYYY-MM-DD--HHMMSS.tar.gz
```

The Moodle data backup is written as:

```bash
SITE_moodledata_backup_YYYY-MM-DD--HHMMSS.tar.gz
```

When `--single-archive` is used, the final archive is:

```bash
SITE_full_backup_YYYY-MM-DD--HHMMSS.tar.gz
```

## Notes

The `moodledata` backup excludes:

* `cache`
* `temp`
* `trashdir`

The site name used in filenames comes from Moodle's course shortname where
`sortorder = 1`. If that query fails, the database name is used instead.

## Compatibility Wrappers

`moodledbbackup.sh` calls `moodle_autoback --db-only`.

`moodlefilesbackup.sh` calls `moodle_autoback --files-only --data-only`.

## Caveats and limitations

I made this tool for my own use (I often say that laziness is the mother of invention, and indeed in this case it is true!) and it's something I use often, so when things go wrong I often find them - however with that said, this is no commercial product by any means and I have not the time to vet all corners of it (especially since updating it so extensively recently, there are bound to be some new sharp edges not yet discovered!). As such, I can in no way guarantee it will be right for anyone else, but all are welcome to use it and my hope is that it helps someone, somewhere, in whatever small way it possibly could. 

Conversely, my advice is if you do indeed use it PLEASE double and triple check that whatever you use it on is restorable from the backups you create from it (pretty standard advice for any backup regime, I know) and do not rely on it as your sole backup process by any means.

One of my favourite use for it is in packing up small to medium Moodles for moving to other servers, which is a nice "safe" way of quickly testing it out, as presumably the original is gonna still be there if the archive has an issue. 

Whatever the case, the upshot is THIS SCRIPT IS PROVIDED ON AN "AS IS" BASIS WITHOUT ANY WARRANTIES OF ANY KIND. BY USING THIS SCRIPT, THE USER CONFIRMS THEY HAVE CONDUCTED THEIR OWN DUE DILIGENCE TO VERIFY ITS SUITABILITY AND SAFETY. THE USER VOLUNTARILY ACCEPTS FULL RISK ASSUMPTION FOR ANY DAMAGES OR DATA LOSS ARISING FROM ITS USE. 

***"Caveat qui exemplaria subsidiaria facit"***
