<h1>Day 10 – File Permissions & File Operations Challenge</h1>

Task
  Master file permissions and basic file operations in Linux.
  
  #) Create and read files using touch, cat, vim
  #) Understand and modify permissions using chmod

Challenge Tasks
  Task 1: Create Files (10 minutes)
  
  Create empty file devops.txt using touch
  Create notes.txt with some content using cat or echo
  Create script.sh using vim with content: echo "Hello DevOps"
  Verify: ls -l to see permissions

  Task 2: Read Files (10 minutes)
    
  Read notes.txt using cat
  View script.sh in vim read-only mode
  Display first 5 lines of /etc/passwd using head
  Display last 5 lines of /etc/passwd using tail


My answer ( Task 1 & 2):

 <img width="760" height="446" alt="Screenshot 2026-03-25 at 9 03 55 PM" src="https://github.com/user-attachments/assets/b6ea7bee-79ee-4add-8652-509f5e61ca06" />

Task 3: Understand Permissions (10 minutes)
  Format: rwxrwxrwx (owner-group-others)
  
  r = read (4), w = write (2), x = execute (1)
  Check your files: ls -l devops.txt notes.txt script.sh


Task 4: Modify Permissions (20 minutes)
  Make script.sh executable → run it with ./script.sh
  Set devops.txt to read-only (remove write for all)
  Set notes.txt to 640 (owner: rw, group: r, others: none)
  Create directory project/ with permissions 755
  Verify: ls -l after each change

Task 5: Test Permissions (10 minutes)
  Try writing to a read-only file - what happens?
  Try executing a file without execute permission
  Document the error messages

My answer ( Task 3,4 & 5):

 <img width="424" height="702" alt="Screenshot 2026-03-25 at 9 09 34 PM" src="https://github.com/user-attachments/assets/c2d3b83f-6ca8-45e3-85f9-d78be580db40" />

Here chmod a-w script.sh represents: chnage permission and remove write permission from all( user + group + others),

a --> all (u + g + o)
u --> user
g --> group
o --> others

r --> read
w --> write
x --> execute

# Summary :

## Files Created
devops.txt  
notes.txt  
script.sh  

## Permission Changes
script.sh → made executable  
devops.txt → read-only  
notes.txt → set to 640  
project/ → set to 755  

## Commands Used
touch, echo, vim, cat, head, tail, chmod, ls  

## What I Learned
- How Linux permissions work (rwx)  
- How to modify permissions using chmod  
- How permissions affect file access and execution  
