---
layout: post
title: CoreOS scheduled tasks
categories:
- coreos
tags:
- coreos
- scheduled task
- cron
- fleet
- timer
- service
- systemd
---

CoreOS uses the Systemd that has built in a bunch of helpful features and tools. For example, instead of using [Cron](https://help.ubuntu.com/community/CronHowto) that is the most common way to run scheduled tasks in most of the Linux distributions, Systemd has the Timer unit configuration that schedules the execution of a Service unit.

This past week I had to configure a scheduled backup routine to run everyday at 2 AM. So this examples is what I had to configure on Systemd to run the backup routine daily. The backup script **backup-mongodb.sh** I have not included in this post because it is just a simple Mongodb backup script.

Basically the simplest way to schedule a service is creating a Timer unit with the same name of the service that is going to be executed by the timer.

## Files

### backup-mongodb.service

It requires the **mongodb.service** is running and it runs the backup script.

```ini
[Unit]
Description=Backup of mongodb
After=mongodb.service
Requires=mongodb.service

[Service]
ExecStart=/bin/bash /home/core/backup-routines/backup-mongodb.sh

[Install]
WantedBy=local.target
```

### backup-mongodb.timer

It also requires the **mongodb.service** is running and it triggers the **backup-mongodb.service** every day at 2 AM. Using **Persistent=true** persists information on last time the timer was triggered. For more information about Timer take a look in the [documentation](http://www.freedesktop.org/software/systemd/man/systemd.timer.html).

```ini
[Unit]
Description=Runs mongodb backup every day at 2 AM
After=mongodb.service
Requires=mongodb.service

[Timer]
OnCalendar=02:00
Persistent=true

[Install]
WantedBy=local.target
```

## Configuring backup schedule

After created those files we just need to load them into the CoreOS machine and start the timer.

```bash
BACKUP_NAME="backup-mongodb"

# removes from coreos these configurations in case they already exist there
fleetctl destroy $(fleetctl list-units | grep "^$BACKUP_NAME")

# loads the service and timer units
fleetctl load $BACKUP_NAME.service
fleetctl load $BACKUP_NAME.timer

# starts the timer
fleetctl start $BACKUP_NAME.timer
```

## Status

Looking the **list-units** you are going to see the timer in **active-waiting** and the service **inactive-dead** status.
Don't worry about the service status. The timer is to start it in the configured time.

```bash
fleetctl list-units | grep backup-mongodb

UNIT                        ACTIVE          SUB
backup-mongodb.service     inactive        dead
backup-mongodb.timer       active          waiting
```
