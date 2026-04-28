# Backup/Reinstall Procedures

## Backup the Database

The VM is configured to create database backups daily. You could access the compressed backups at `var/backups/mysql`.
The backups will each last for one month.

You could choose a backup, unzip it, and then import it into MySQL.
```bash
gunzip < saln_app_DB_MM_DD_YYYY.sql.gz > saln_app_DB_MM_DD_YYYY.sql
mysql -u <USER> -p saln_app_DB < saln_app_DB_MM_DD_YYYY.sql
```