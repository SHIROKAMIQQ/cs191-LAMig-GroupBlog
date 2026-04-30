# Backup/Reinstall Procedures

## Backup the Database

The VM is configured to create database backups daily. You could access the compressed backups at `var/backups/mysql`.
The backups will each last for one month.

You could choose a backup, unzip it, and then import it into MySQL.
```bash
gunzip < saln_app_DB_YYYY_MM_DD.sql.gz > saln_app_DB_YYYY_MM_DD.sql
mysql -u <USER> -p saln_app_DB < saln_app_DB_YYYY_MM_DD.sql
```

## App Backup/Restore Procedures

### Source Code Restoration

This repository uses Git for version control. In the event that you want to branch off from a different commit, you could checkout that commit on a different branch from main, make your edits (or just use that version as is), and then merge it into the main branch. 
```bash
# Get commit hash of version you want
git log --oneline

# Create new bracnh starting from that commit
git checkout -b <featureBranch> <commitHash>

# MAKE CHANGES (OR JUST USE AS IS)

# Rebase onto main
git checkout <featureBranch>
git rebase main
git checkout main
git merge <featureBranch>
```

### .env Backup and Restoration Procedure

`saln-server/.env` is not part of the Git respository since it contains secret keys. So instead, we will keep backups on the virtual machine accessible only via root. You may check out [.env Backup/Restore Setup](DOSetup.md#env-backuprestore-setup) to have a better idea of what's going on.

Create a backup of the current `.env` file:
```bash
env_backup
```

This creates a backup with an associated timestamp on its file name. You could check out your backups via
```bash
ls -a /var/backups/saln-env
```

To restore a specific backup:
```bash
env_restore YYYY_MM_DD_HHMMSS
```
