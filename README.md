# Nextcloud-Backup-Restore

This repository contains two bash scripts for the backup and/or restore of [Nextcloud](https://nextcloud.com/).

It is based on a standard Nextcloud-installation using Apache2 and PostgreSQL but works well with nginx as webserver and MariaDB/MySQL as database. The original can be found at [Decatec's Codeberg](https://codeberg.org/DecaTec/Nextcloud-Backup-Restore/)-page and credits go to Decatec for the initial creation of this script!

## General information

For a complete backup of any existing Nextcloud-instance, you'll have to backup these items:

- The Nextcloud file directory (usually */var/www/nextcloud*)
- The data directory of Nextcloud (it's recommended that this is *not* located in the web root, so e.g. */var/nextcloud_data*)
- The Nextcloud-database
- Maybe a local external storage mounted into Nextcloud (Wasabi's S3-storage does a great job here)

The scripts take care of these items to backup automatically.

**Important:**

- After cloning or downloading the repository, you'll have to edit the scripts so that they represent your current and individual Nextcloud-installation (directories, users, etc.). All values which need to be customized are marked with *TODO* in the script's comments
- The scripts assume that Nextcloud's data directory is *not* a subdirectory of the Nextcloud installation (file directory). The general recommendation is that the data directory should not be located somewhere in the web folder of your webserver (usually */var/www/*), but in a different folder (e.g. */var/nextcloud_data*). For more information, see [here](https://docs.nextcloud.com/server/latest/admin_manual/installation/installation_wizard.html#data-directory-location-label)
- However, if your data directory *is* located under the Nextcloud file directory, you'll have to change the scripts so that the data directory is not part of the backup/restore (otherwise, it would be copied twice)
- The scripts only backup the Nextcloud data directory and can backup a local external storage mounted into Nextcloud. If you have any other external storage mounted in Nextcloud (e.g. FTP, S3), these files have to be handled separately
- The scripts assume that, in my specific case, PostgreSQL is used as database for Nextcloud. However, it also supports the standard MySQL/MariaDBL-databases. In this case you have to uncomment the parts of backing up/restoring the database
- You should have enabled 4 byte support (see [Nextcloud Administration Manual](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/mysql_4byte_support.html)) on your Nextcloud database. Otherwise, when you have *not* enabled 4 byte support, you will have to edit the restore script so that the database is not created with 4 byte support enabled

## Backup

In order to create a backup, simply call the script *NextcloudBackup.sh* on your Nextcloud-machine.
If this script is called without parameter, the backup is saved in a directory with the current time stamp in your main backup directory: As an example, this would be */media/hdd/nextcloud_backup/20210627_222645*.

The backup script can also be called with a parameter specifiying the main backup directory, e.g. *./NextcloudBackup.sh /media/hdd/nextcloud_backup*. In this case, the directory specified will be used as main backup directory. 

You can also call this script by using cron. In this example, it runs at 2am every night with log output:

`0 2 * * * /path/to/scripts/Nextcloud-Backup-Restore/NextcloudBackup.sh  > /path/to/logs/Nextcloud-Backup-$(date +\%Y\%m\%d\%H\%M\%S).log 2>&1`

## Restore

For restore, just call *NextcloudRestore.sh*. This script expects at least one parameter specifying the name of the backup to be restored. In our example, this would be *20210627_222645* (the time stamp of the backup created before).

The full command for a restore would be *./NextcloudRestore.sh 20210627_222645*. You can also specify the main backup directory with a second parameter, e.g. *./NextcloudRestore.sh 20210627_222645 /media/hdd/nextcloud_backup*.

## Automated setup

Next to the backup/restore scripts, there is another script (`setup.sh`). The setup script gathers some information and uses the [OCC command](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html) in order to set the required variables in the backup/restore-scripts automatically. Going this way, the configuration of the backup/restore-scripts can be automated to some extend.
