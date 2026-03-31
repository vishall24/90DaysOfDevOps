# Day 16 – Shell Scripting Basics

## Task
Start shell scripting journey — learn the fundamentals every script needs.

I will:
- Understand **shebang** (`#!/bin/bash`) and why it matters
- Work with **variables**, **echo**, and **read**
- Write basic **if-else** conditions


## Challenge Tasks

### Task 1: Your First Script
1. Create a file `hello.sh`
2. Add the shebang line `#!/bin/bash` at the top
3. Print `Hello, DevOps!` using `echo`
4. Make it executable and run it

======

My answer:


<img width="519" height="167" alt="Screenshot 2026-03-31 at 10 01 44 PM" src="https://github.com/user-attachments/assets/0ddf7d57-4bc4-445f-8344-b26b1bd976a9" />

 Q) **Document:** What happens if you remove the shebang line? 

    So to answer this, I have removed the shebang and it still did work, why? because it is using default shell that is (sh) and sh does
    support the syntax that I have written,

     then why shebang buddy ? actually lets say you run a script but that script is written for ( bash ) script and some comand or keyword 
     may not work with ( sh ). thats why we explicitly mention which bash to use to the interpreter of the system, if not mentioned any, then
     it will consider the default interpreter ( sh ).


### Task 2: Variables

1. Create `variables.sh` with:
   - A variable for your `NAME`
   - A variable for your `ROLE` (e.g., "DevOps Engineer")
   - Print: `Hello, I am <NAME> and I am a <ROLE>`
2. Try using single quotes vs double quotes — what's the difference?


=====

My answer:


  <img width="1196" height="312" alt="image" src="https://github.com/user-attachments/assets/4544a763-5c83-476a-b9c4-a29d03ade21d" />
  
  the difference between using single vs double qoutes is present in the screenshot above.


### Task 3: User Input with read
1. Create `greet.sh` that:
   - Asks the user for their name using `read`
   - Asks for their favourite tool
   - Prints: `Hello <name>, your favourite tool is <tool>`

======

My answer:


<img width="1256" height="232" alt="image" src="https://github.com/user-attachments/assets/f99f31d2-c747-49fc-bb6a-41ae54e71e41" />


<img width="1216" height="450" alt="image" src="https://github.com/user-attachments/assets/e093851e-0e2f-4f92-b071-3e7e790d7d0a" />


here -p = print a message before reading input


### Task 4: If-Else Conditions

1. Create `check_number.sh` that:
   - Takes a number using `read`
   - Prints whether it is **positive**, **negative**, or **zero**

2. Create `file_check.sh` that:
   - Asks for a filename
   - Checks if the file **exists** using `-f`
   - Prints appropriate message

=======


My answer:

for point ( 1 ):


<img width="1192" height="460" alt="image" src="https://github.com/user-attachments/assets/3dc99195-b031-4c00-9181-0d3a30009728" />


<img width="423" height="301" alt="Screenshot 2026-03-31 at 10 34 27 PM" src="https://github.com/user-attachments/assets/dc55b3a3-33ab-436c-b214-45a1a52e718a" />


for point ( 2 ):


<img width="1200" height="666" alt="image" src="https://github.com/user-attachments/assets/463a4561-739c-40c8-951f-a50f09248c72" />


<img width="1446" height="390" alt="image" src="https://github.com/user-attachments/assets/2a6b6c1b-a03b-4173-bf62-ad55ab30d102" />


### Task 5: Combine It All
Create `server_check.sh` that:
1. Stores a service name in a variable (e.g., `nginx`, `sshd`)
2. Asks the user: "Do you want to check the status? (y/n)"
3. If `y` — runs `systemctl status <service>` and prints whether it's **active** or **not**
4. If `n` — prints "Skipped."


======

My answer:


<img width="2786" height="846" alt="image" src="https://github.com/user-attachments/assets/6948ce6b-d461-4e4b-b699-967d71d4873d" />


<img width="1130" height="242" alt="image" src="https://github.com/user-attachments/assets/3a39785e-33ff-4070-9957-9c969b811839" />


## Summary:

## Scripts Created

hello.sh  
variables.sh  
greet.sh  
check_number.sh  
file_check.sh  
server_check.sh  

## What I Learned
- How to write basic shell scripts  
- How to use variables
- How to take user input  
- How to use if-else conditions

  

  
