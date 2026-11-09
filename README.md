# PostgreSQL Lab: Backups, PITR, and Replication

## Step 1: Logical Backup
```bash
mkdir -p ~/backups
pg_dump -Fc -f ~/backups/bootcamp.dump bootcamp
createdb bootcamp_check && pg_restore -d bootcamp_check ~/backups/bootcamp.dump

# Configured postgresql.conf with:
wal_level = replica
archive_mode = on
archive_command = 'cp %p /home/$USER/backups/wal/%f'

mkdir -p ~/backups/wal
sudo systemctl restart postgresql
pg_basebackup -D ~/backups/base -Ft -z -Xs -P

#Point-in-Time Recovery (PITR)
Recorded target recovery time and simulated disaster using DELETE FROM students;.

Restored base backup, configured restore_command, and set recovery_target_time
