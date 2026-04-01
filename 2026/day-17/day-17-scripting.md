# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Task
Level up your scripting — use loops, handle arguments, and deal with errors.

You will:
- Write **for** and **while** loops
- Use **command-line arguments** (`$1`, `$2`, `$#`, `$@`)
- Install packages via script
- Add basic **error handling**

---

## Challenge Tasks

### Task 1: For Loop
1. Create `for_loop.sh` that:
   - Loops through a list of 5 fruits and prints each one
2. Create `count.sh` that:
   - Prints numbers 1 to 10 using a for loop

---

My answer : 

<img width="1172" height="464" alt="image" src="https://github.com/user-attachments/assets/357973e6-5a63-4573-b4fe-7a25fb63e84a" />

<img width="1324" height="448" alt="image" src="https://github.com/user-attachments/assets/1617aa7a-ca09-4a0d-a557-ae88ea7441f1" />

<img width="1028" height="566" alt="image" src="https://github.com/user-attachments/assets/d798c6ad-25cd-4257-8884-ea73ae09a734" />

<img width="568" height="588" alt="image" src="https://github.com/user-attachments/assets/7a509e3b-32fd-46df-9daa-d77e63cb3628" />

---

### Task 2: While Loop
1. Create `countdown.sh` that:
   - Takes a number from the user
   - Counts down to 0 using a while loop
   - Prints "Done!" at the end

---

My answer:

<img width="964" height="648" alt="image" src="https://github.com/user-attachments/assets/e316d624-375a-41b9-a17f-047b7fc25ab5" />

<img width="780" height="610" alt="image" src="https://github.com/user-attachments/assets/76f45605-3a02-4d98-88d2-0218e57805c7" />

---

### Task 3: Command-Line Arguments
1. Create `greet.sh` that:
   - Accepts a name as `$1`
   - Prints `Hello, <name>!`
   - If no argument is passed, prints "Usage: ./greet.sh <name>"

2. Create `args_demo.sh` that:
   - Prints total number of arguments (`$#`)
   - Prints all arguments (`$@`)
   - Prints the script name (`$0`)

---

My answer:

<img width="564" height="248" alt="image" src="https://github.com/user-attachments/assets/26281dcc-5ab9-4e33-8dc9-106a81515da5" />

<img width="1166" height="152" alt="image" src="https://github.com/user-attachments/assets/8906d517-242a-4552-835c-5d48e3d9e2ec" />

<img width="1076" height="240" alt="image" src="https://github.com/user-attachments/assets/55cdcd14-3b04-42c5-9c65-cc588795811f" />

<img width="790" height="576" alt="image" src="https://github.com/user-attachments/assets/a5b620fe-55f9-47de-a6ed-d61603e24ecd" />

---

### Task 4: Install Packages via Script
1. Create `install_packages.sh` that:
   - Defines a list of packages: `nginx`, `curl`, `wget`
   - Loops through the list
   - Checks if each package is installed (use `dpkg -s` or `rpm -q`)
   - Installs it if missing, skips if already present
   - Prints status for each package

> Run as root: `sudo -i` or `sudo su`

---

My answer:

<img width="1192" height="222" alt="image" src="https://github.com/user-attachments/assets/cd644f58-8282-48c0-bf66-40cad2b7569c" />

<img width="1176" height="792" alt="image" src="https://github.com/user-attachments/assets/6fd2ae82-8949-49b0-af4a-1ac87afb1112" />

Here there was a mistake in the script: it should be $pkgs instead of $pkg in if condition

---


### Task 5: Error Handling
1. Create `safe_script.sh` that:
   - Uses `set -e` at the top (exit on error)
   - Tries to create a directory `/tmp/devops-test`
   - Tries to navigate into it
   - Creates a file inside
   - Uses `||` operator to print an error if any step fails

Example:
```bash
mkdir /tmp/devops-test || echo "Directory already exists"
```

2. Modify your `install_packages.sh` to check if the script is being run as root — exit with a message if not.

---



<img width="1582" height="150" alt="image" src="https://github.com/user-attachments/assets/377bb80f-24b6-449b-a6cb-68ed440809a1" />



<img width="1530" height="508" alt="image" src="https://github.com/user-attachments/assets/dfc7c4be-b610-4238-8f5a-116479e87a62" />



<img width="1132" height="116" alt="image" src="https://github.com/user-attachments/assets/35c5c6b0-3659-48e9-9de3-3d8161862124" />



<img width="1140" height="1008" alt="image" src="https://github.com/user-attachments/assets/1cff6972-ae84-44e5-b4c7-1063ca136351" />

---

## Scripts Created

    for_loop.sh  
    count.sh  
    countdown.sh  
    greet.sh  
    args_demo.sh  
    install_packages.sh  
    safe_script.sh  

## What I Learned

    - How to use loops in scripting  
    - How to pass arguments to scripts  
    - Basic automation and error handling  
