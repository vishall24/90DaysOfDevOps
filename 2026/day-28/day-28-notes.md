# Day 28 – Revision Day: Everything from Day 1 to Day 27

## Task

---

## What I have Covered So Far

| Days | Topic | Key Concepts |
|------|-------|-------------|
| 1 | DevOps & Cloud Intro | What is DevOps, SDLC, Cloud basics |
| 2–7 | Linux Fundamentals | Architecture, commands, processes, systemd, file system hierarchy, troubleshooting, text files |
| 8 | Cloud Server Setup | Docker, Nginx, web deployment |
| 9–11 | Users, Permissions & Ownership | User/group management, file permissions, chown/chgrp |
| 12 | Revision Day 1 | Days 1–11 recap |
| 13 | Volume Management | LVM — physical volumes, volume groups, logical volumes |
| 14–15 | Networking | Fundamentals, DNS, IP, subnets, ports, hands-on checks |
| 16–18 | Shell Scripting | Basics, loops, arguments, error handling, functions |
| 19–20 | Shell Scripting Projects | Log rotation, backup, crontab, log analyzer |
| 21 | Shell Scripting Cheat Sheet | Personal reference guide |
| 22–25 | Git & GitHub | Init, branching, merge, rebase, stash, cherry pick, reset, revert, branching strategies |
| 26 | GitHub CLI | Managing GitHub from the terminal |
| 27 | GitHub Profile | Profile README, repo organization, developer branding |

---

#### Linux
- [x] Navigate the file system, create/move/delete files and directories
- [x] Manage processes — list, kill, background/foreground
- [x] Work with systemd — start, stop, enable, check status of services
- [x] Read and edit text files using vi/vim or nano
- [x] Troubleshoot CPU, memory, and disk issues using top, free, df, du
- [x] Explain the Linux file system hierarchy (/, /etc, /var, /home, /tmp, etc.)
- [x] Create users and groups, manage passwords
- [x] Set file permissions using chmod (numeric and symbolic)
- [x] Change file ownership with chown and chgrp
- [ ] Create and manage LVM volumes
- [x] Check network connectivity — ping, curl, netstat, ss, dig, nslookup
- [x] Explain DNS resolution, IP addressing, subnets, and common ports

#### Shell Scripting
- [x] Write a script with variables, arguments, and user input
- [x] Use if/elif/else and case statements
- [x] Write for, while, and until loops
- [x] Define and call functions with arguments and return values
- [x] Use grep, awk, sed, sort, uniq for text processing
- [x] Handle errors with set -e, set -u, set -o pipefail, trap
- [x] Schedule scripts with crontab

#### Git & GitHub
- [x] Initialize a repo, stage, commit, and view history
- [x] Create and switch branches
- [x] Push to and pull from GitHub
- [x] Explain clone vs fork
- [x] Merge branches — understand fast-forward vs merge commit
- [x] Rebase a branch and explain when to use it vs merge
- [x] Use git stash and git stash pop
- [x] Cherry-pick a commit from another branch
- [x] Explain squash merge vs regular merge
- [x] Use git reset (soft, mixed, hard) and git revert
- [ ] Explain GitFlow, GitHub Flow, and Trunk-Based Development
- [x] Use GitHub CLI to create repos, PRs, and issues

---


1. LVM:

   Revisited LVM day 13th md file and got to know how to create and manage LVM , steps:

   lsblk -> list all blocks

   pvcreate /dev/you_storage_name -> find your newly created Storage and create a physical Volume for it.

   pvs -> to check the physical volumes

   vgcreate storage-vg /dev/storage -> create volume group

   vgs -> list all volume groups

   lgcreate -L 2G -n app-data storage-vg -> this will create logical volume of size 2G under the volume group storage-vg & the logical volume name is app-data

   mkfs.ext4 /dev/devops-vg/app-data -> this will create a filesystem to do more operation on that

   mkdir /tmp/data -> creates dir /tmp/data

   mount /dev/devops-vg/app-data /tmp/data --> this will mount the storage /dev/devops-vg/app-data to /tmp/data and now the path /tmp/data has 2G of storage.
   
2. Explain GitFlow, GitHub Flow, and Trunk-Based Development:

      GitFlow — develop, feature, release, hotfix branches

      GitHub Flow — simple, single main branch + feature branches

      Trunk-Based Development — everyone commits to main, short-lived branches

    What to use when:

    GitFlow
    Flow:
         
         main → production  
         develop → working  
         feature → new work  
         release → prepare release  
         hotfix → urgent fix  
   Use:
         
         big teams
         scheduled releases
   
   GitHub Flow
   Flow:
         
         main → always deployable  
         feature branch → PR → merge  
    Use:
         
         startups
         fast deployment
    Trunk-Based
    Flow:
         
         main → everyone commits
   
    short-lived branches  
    Use:
         
         high-speed teams
         CI/CD heavy
         ANSWERS
         
     Startup:
         
          GitHub Flow
     Large team:
         
          GitFlow
     Modern DevOps:

         Trunk-Based

### Task 3: Quick-Fire Questions
Answer these from memory (no Googling). Then verify your answers:

1. What does `chmod 755 script.sh` do?
2. What is the difference between a process and a service?
3. How do you find which process is using port 8080?
4. What does `set -euo pipefail` do in a shell script?
5. What is the difference between `git reset --hard` and `git revert`?
6. What branching strategy would you recommend for a team of 5 developers shipping weekly?
7. What does `git stash` do and when would you use it?
8. How do you schedule a script to run every day at 3 AM?
9. What is the difference between `git fetch` and `git pull`?
10. What is LVM and why would you use it instead of regular partitions?

---

My answer:

1. it sets the permission read/write/execute --> owner , read/execute --> groups , read/execute --> others to the file script.sh

2. A process is a task or job while service is kind of application which can generate logs as well. ( actual ans: Process = running program, Service = long-running background process (managed by system))

3. Maybe tulnip command which shows ports and their service name as well. ( actual ans: " lsof -i :8080 " OR  " ss -tulpn | grep 8080 ")

4. -e means if there is any error exit, pipeline means if there is any error in the pipeline operator ( | ) then exit ( actual ans: -u exit on unset vars, -o is long option eg: -o <option_name>, -o pipeline).

5. git reset --hard will reset the commit ID and it removes the commit history as well as local changes, git revert will revert the changes but creates a new commit ID, also keep the commit history.
   
6. gitHub flow
   
7. git stash will stash the changes, useful when working with multiple branches and dont want one branch change in another
    
8. 0 3 * * * this will schedule the cron.
    
9. git fetch vs git pull --> fetch -> download only  ,,,  pull -> fetch + merge
    
10. LVM gives as the freedom to use the storage dynamically, used if traffic increases it then can increase the storage if configured.

### Task 5: Teach It Back

  My answer:

File permision:

What is file permission? as the name itself says it manages the permission of file and in linux we can edit the permissions as well:
ever heard of chmod 755 script.sh ?? 

let me explain what is this

here chmod is change mode , and 755??

here is the breakdowm of 755:

7 --> means read & write & execute, how?? remember:

    4 --> read
    2 --> write
    1 --> execute

    EX: 6 = 4+2 == R/W, 3 = 1+2 == X/W

at the last , there is a file name , BTW it applies to folder as well , so yes, you can use them for folder as well.
