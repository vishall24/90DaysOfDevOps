<h1>Day 11 – File Ownership Challenge (chown & chgrp)</h1>

Task:
  Master file and directory ownership in Linux.

  Understand file ownership (user and group)
  Change file owner using chown
  Change file group using chgrp
  Apply ownership changes recursively

Challenge Tasks

<h2>Task 1: Understanding Ownership</h2>
  
  Run ls -l in your home directory
  Identify the owner and group columns
  Check who owns your files
  Format: -rw-r--r-- 1 owner group size date filename
  
  Document: What's the difference between owner and group?
My answer:

<img width="964" height="302" alt="image" src="https://github.com/user-attachments/assets/ff185947-a321-48cd-8874-2ede21235fed" />

Here Ubuntu is the User/Owner and another ubuntu right after the first ubuntu is the group.

Owner --> who controls file
Groups --> shared access for pool of users.

<h2>Task 2: Basic chown Operations</h2>

  Create file devops-file.txt
  Check current owner: ls -l devops-file.txt
  Change owner to tokyo (create user if needed)
  Change owner to berlin
  Verify the changes
  
My answer:

<img width="1038" height="1408" alt="image" src="https://github.com/user-attachments/assets/c41f91a2-6f1f-4087-a937-3c0b80c4c6c4" />

<h2>Task 3: Basic chgrp Operations</h2>
  
  Create file team-notes.txt
  Check current group: ls -l team-notes.txt
  Create group: sudo groupadd heist-team
  Change file group to heist-team
  Verify the change

My answer:

 <img width="894" height="194" alt="image" src="https://github.com/user-attachments/assets/2fab3bc9-f822-4cac-b967-0a70bf4a37ca" />

<h2>Task 4: Combined Owner & Group Change</h2>
  
  Using chown you can change both owner and group together:
  
  Create file project-config.yaml
  Change owner to professor AND group to heist-team (one command) --> can use "" chown professor:heist-team file_name ""
  Create directory app-logs/
  Change its owner to berlin and group to heist-team

My answer:

<img width="1108" height="826" alt="image" src="https://github.com/user-attachments/assets/d0b77f14-2a7a-4283-8987-1e0f1774a2de" />

<h2>Task 5: Recursive Ownership</h2>
  
  Create directory structure:
  
  mkdir -p heist-project/vault
  mkdir -p heist-project/plans
  touch heist-project/vault/gold.txt
  touch heist-project/plans/strategy.conf
  Create group planners: sudo groupadd planners
  
  Change ownership of entire heist-project/ directory:
  
  Owner: professor
  Group: planners
  Use recursive flag (-R)
  Verify all files and subdirectories changed: ls -lR heist-project/

My answer:

<img width="1140" height="1454" alt="image" src="https://github.com/user-attachments/assets/a274d78f-5e57-4fda-8229-8336faa13d3c" />

<h2>Task 6: Practice Challenge</h2>
  
  Create users: tokyo, berlin, nairobi (if not already created)
  
  Create groups: vault-team, tech-team
  
  Create directory: bank-heist/
  
  Create 3 files inside:
  
  touch bank-heist/access-codes.txt
  touch bank-heist/blueprints.pdf
  touch bank-heist/escape-plan.txt
  Set different ownership:
  
  access-codes.txt → owner: tokyo, group: vault-team
  blueprints.pdf → owner: berlin, group: tech-team
  escape-plan.txt → owner: nairobi, group: vault-team
  Verify: ls -l bank-heist/

My answer:

<img width="1236" height="532" alt="image" src="https://github.com/user-attachments/assets/3ebb37fb-1c98-4972-8470-91b36de6f656" />


# Summary

## Files & Directories Created
  devops-file.txt  
  team-notes.txt  
  project-config.yaml  
  heist-project/  
  bank-heist/  

## Ownership Changes
  devops-file.txt → tokyo → berlin  
  team-notes.txt → group changed to heist-team  
  project-config.yaml → professor:heist-team  
  heist-project → professor:planners (recursive)  

## Commands Used
  chown, chgrp, ls, mkdir, touch  

## What I Learned
  - Difference between owner and group  
  - How to change ownership using chown  
  - How recursive ownership works  

Cias Adios :)
