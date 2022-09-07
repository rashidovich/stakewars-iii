auto-backup feature consist of 3 separate things
- cron job 
- backup script
- backup restore script

cron job will execute backup script on some schedule, how regular depends on you preferences.
lets assume `backup.sh` script loction is in /home/near directory and you want to have a backup of near database once per day then cron job would look like
`0 0 * * * /home/near/backup.sh`

backup script that packs db into archive, stores archive in /home/near/backups dir and deletes previous/older backups
```
#!/bin/bash

DATE=$(date +%Y-%m-%d-%H-%M)
DATADIR=/home/near
BACKUPDIR=${DATADIR}/backups/
ARCHIVE_NAME=near_${DATE}
ARCHIVEDIR=${BACKUPDIR}/${ARCHIVE_NAME}

mkdir ${ARCHIVEDIR}

sudo systemctl stop near.service

wait

echo "NEAR node was stopped" | ts

if [ -d "$ARCHIVEDIR" ]; then
    echo "Backup started" | ts

    cp -rf $DATADIR/.near/data/ ${ARCHIVEDIR}
    zip -rjD ${ARCHIVEDIR}.zip ${ARCHIVEDIR}

    echo "Backup completed" | ts
    echo "Removing previous backups:" | ts

	  find ${BACKUPDIR} -type f -not -name ${ARCHIVE_NAME}.zip -print0 | xargs -0  -I {} rm -v {}
else
    echo $BACKUPDIR is not created. Check your permissions.
    exit 0
fi

sudo systemctl start near.service

echo "NEAR node was started" | ts
```
to restore db in automated fashion use `restore.sh` script
```
#!/bin/bash

DATE=$(date +%Y-%m-%d-%H-%M)
DATADIR=/home/near
BACKUPDIR=${DATADIR}/backups
BACKUP_NAME=near_${DATE}

sudo systemctl stop near.service

wait

echo "NEAR node was stopped"
echo "Restoring backup"

backup_path=`ls -t ${BACKUPDIR}/*.zip | sort | head -1`
echo "Backup: ${backup_path}"

unzip -o ${backup_path} -d ${DATADIR}/.near/data

sudo systemctl start near.service

echo "Backup was restored
echo "NEAR node was started"
```
