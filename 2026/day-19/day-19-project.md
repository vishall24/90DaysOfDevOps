# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Task
    
      Apply everything from Days 16–18 in real-world mini projects.
      
      I will:
      - Write a **log rotation** script
      - Write a **server backup** script
      - Schedule them with **crontab**

---

## Challenge Tasks

### Task 1: Log Rotation Script

    Create `log_rotate.sh` that:
    1. Takes a log directory as an argument (e.g., `/var/log/myapp`)
    2. Compresses `.log` files older than 7 days using `gzip`
    3. Deletes `.gz` files older than 30 days
    4. Prints how many files were compressed and deleted
    5. Exits with an error if the directory doesn't exist

---

My answer:

<img width="1544" height="226" alt="image" src="https://github.com/user-attachments/assets/aafa9360-6a31-4c65-8474-7e9c6cfb6bb9" />

<img width="1704" height="846" alt="image" src="https://github.com/user-attachments/assets/70a5feaa-ef5c-496c-8118-7a70fa46a362" />

---

### Task 2: Server Backup Script
Create `backup.sh` that:
1. Takes a source directory and backup destination as arguments
2. Creates a timestamped `.tar.gz` archive (e.g., `backup-2026-02-08.tar.gz`)
3. Verifies the archive was created successfully
4. Prints archive name and size
5. Deletes backups older than 14 days from the destination
6. Handles errors — exit if source doesn't exist

---

My answer:

<img width="1780" height="244" alt="image" src="https://github.com/user-attachments/assets/1b703931-6033-4342-837b-243652b90a7a" />

<img width="1468" height="1252" alt="image" src="https://github.com/user-attachments/assets/006dee91-a73a-4529-951a-bb577ef7bf4f" />

---

### Task 3: Crontab
    1. Read: `crontab -l` — what's currently scheduled?
    2. Understand cron syntax:
       ```
       * * * * *  command
       │ │ │ │ │
       │ │ │ │ └── Day of week (0-7)
       │ │ │ └──── Month (1-12)
       │ │ └────── Day of month (1-31)
       │ └──────── Hour (0-23)
       └────────── Minute (0-59)
       ```
    3. Write cron entries (in your markdown, don't apply if unsure) for:
       - Run `log_rotate.sh` every day at 2 AM
       - Run `backup.sh` every Sunday at 3 AM
       - Run a health check script every 5 minutes

---

My answer:

<img width="1012" height="138" alt="image" src="https://github.com/user-attachments/assets/6b9fd6a7-4b41-48d9-aceb-91126eede171" />

    # daily at 2 AM
    * 2 * * * ./log_rotation.sh /var/log/myapps
    
    # daily at 3 AM
    * 3 * * * ./backup.sh /tmp/source-data /tmp/backups
    
    # Runs health check script every 5 minutes
    */5 * * * * ./health_check.sh

---

### Task 4: Combine — Scheduled Maintenance Script
Create `maintenance.sh` that:
1. Calls your log rotation function
2. Calls your backup function
3. Logs all output to `/var/log/maintenance.log` with timestamps
4. Write the cron entry to run it daily at 1 AM

---

My answer:

<img width="1476" height="592" alt="image" src="https://github.com/user-attachments/assets/a0c34b52-40c4-4e11-98bd-0d130a01d454" />

<img width="1518" height="458" alt="image" src="https://github.com/user-attachments/assets/36e03e81-76e2-4bf1-b290-887c9b3783c3" />

## Scripts Created
    log_rotate.sh  
    backup.sh  
    maintenance.sh  

## Cron Jobs
       * * * * *  command
       │ │ │ │ │
       │ │ │ │ └── Day of week (0-7)
       │ │ │ └──── Month (1-12)
       │ │ └────── Day of month (1-31)
       │ └──────── Hour (0-23)
       └────────── Minute (0-59)

       Edited cron job by executing:: crontab -e, and edited in vim editor 

## What I Learned
    - Automating real tasks like logs and backups  
    - Scheduling jobs using cron  
    - Writing production-like scripts  


Ciao Adios
