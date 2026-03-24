<h1>Day 09 – Linux User & Group Management Challenge</h1>
<h2>Task</h2>
  Today's goal is to practice user and group management by completing hands-on challenges.
  
  Figure out how to:
  
  Create users and set passwords
  Create groups and assign users
  Set up shared directories with group permissions

Task 1: Create Users:

  Create three users with home directories and passwords:
  
  tokyo
  berlin
  professor

  Verify: Check /etc/passwd and /home/ directory

My answer:

 Executed these command for creatig users:

 - sudo useradd -m tokyo
 - sudo useradd -m berlin
 - sudoo useradd -m professor

Here -m is used for creating home/tokyo (home dir for mentioned user) directory 

Executed these command for setting password:

<img width="497" height="289" alt="Screenshot 2026-03-24 at 11 31 39 PM" src="https://github.com/user-attachments/assets/f62e619b-cc24-4d66-accd-e762bc34f978" />

verified with executing :: cat /etc/passwd && ls /home

Task 2: Create Groups:

 Create two groups:

  developers
  admins
  
  Verify: Check /etc/group

My answer:

Executed these commands to create groups and verified:

- sudo groupadd developers
- sudo groupadd admins

Verified them using : cat /etc/group | grep "developers"


Task 3: Assign to Groups (15 minutes)
Assign users:

tokyo → developers
berlin → developers + admins (both groups)
professor → admins
Verify: Use appropriate command to check group membership

My answer:

  Executed these commands to assign users to respective group:

  <img width="1300" height="474" alt="image" src="https://github.com/user-attachments/assets/0979cec2-374a-43b1-a0f9-2a0756e77a10" />


Task 4: Shared Directoryd
  
  Create directory: /opt/dev-project
  Set group owner to developers
  Set permissions to 775 (rwxrwxr-x)
  Test by creating files as tokyo and berlin
  Verify: Check permissions and test file creation

My answer:

  <img width="1540" height="582" alt="image" src="https://github.com/user-attachments/assets/8095a4be-9442-400a-ae6a-cdf490f38837" />


Task 5: Team Workspace (20 minutes)
  
  Create user nairobi with home directory
  Create group project-team
  Add nairobi and tokyo to project-team
  Create /opt/team-workspace directory
  Set group to project-team, permissions to 775
  Test by creating file as nairobi

My answer:

  <img width="1660" height="750" alt="image" src="https://github.com/user-attachments/assets/d656e70c-ef11-45fa-8fb7-213d56a28151" />


## Summary :

## Users & Groups Created
Users: tokyo, berlin, professor, nairobi  
Groups: developers, admins, project-team  

## Group Assignments
tokyo → developers, project-team  
berlin → developers, admins  
professor → admins  
nairobi → project-team  

## Directories Created
/opt/dev-project → developers group, 775  
/opt/team-workspace → project-team group, 775  

## Commands Used
useradd, passwd, groupadd, usermod, chgrp, chmod  

## What I Learned
- How to manage users and groups  
- How permissions control access  
- How shared directories work in real systems

Ciao Adios :)
