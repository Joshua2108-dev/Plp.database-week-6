#  backup directory
mkdir -p ~/backups

#  custom-format logical dump
pg_dump -Fc -f ~/backups/bootcamp.dump bootcamp

# A check database and restored the backup
createdb bootcamp_check 
pg_restore -d bootcamp_check ~/backups/bootcamp.dump

psql -d bootcamp_check -c "SELECT count(*) FROM students;"count 
-------
   150
(1 row)

Enable WAL Archiving and Base Backup
Configure PostgreSQL for continuous archiving and take a physical base backup.
wal_level = replica
archive_mode = on
archive_command = 'cp %p /home/$USER/backups/wal/%f'
mkdir -p ~/backups/wal
sudo systemctl restart postgresql
pg_basebackup -D ~/backups/base -Ft -z -Xs -P

Simulate the Disaster
Log into PostgreSQL, record the timestamp, and accidentally delete the data:

SELECT now();   
-- Recorded timestamp: 2025-06-01 10:00:00

DELETE FROM students;   
-- Oops! All student records deleted.

SELECT count(*) FROM students;
count 
-------
     0
(1 row)

Execute Recovery Steps
Stop PostgreSQL:

sudo systemctl stop postgresql
sudo rm -rf /var/lib/postgresql/15/main/*
sudo tar -xzf ~/backups/base/base.tar.gz -C /var/lib/postgresql/15/main/

echo "restore_command = 'cp /home/$USER/backups/wal/%f %p'" | sudo tee -a /var/lib/postgresql/15/main/postgresql.conf
echo "recovery_target_time = '2025-06-01 10:00:00'" | sudo tee -a /var/lib/postgresql/15/main/postgresql.conf

sudo systemctl start postgresql
psql -d bootcamp -c "SELECT count(*) FROM students;"
count 
-------
   150
(1 row)

Set Up a Streaming Standby Replica
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'reppass';
pg_basebackup -h 127.0.0.1 -U replicator -D ~/standby -R -P

Watch Replication Health
psql -d bootcamp -c "SELECT application_name, state, pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes FROM pg_stat_replication;"
Query Output:
application_name |   state   | lag_bytes ------------------+-----------+-----------
 walreceiver      | streaming |         0
(1 row)


