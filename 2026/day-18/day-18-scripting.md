# Day 18 – Shell Scripting: Functions & intermediate Concepts

## Task
Write cleaner, reusable scripts — learn functions, strict mode, and real-world patterns.

You will:
- Write and call **functions**
- Use **`set -euo pipefail`** for safer scripts
- Work with **return values** and **local variables**
- Build an intermediate script

---

## Challenge Tasks

### Task 1: Basic Functions
1. Create `functions.sh` with:
   - A function `greet` that takes a name as argument and prints `Hello, <name>!`
   - A function `add` that takes two numbers and prints their sum
   - Call both functions from the script

---

My answer:

<img width="484" height="102" alt="Screenshot 2026-04-02 153927" src="https://github.com/user-attachments/assets/e0bdb858-56a5-4714-8005-e3b97bcf4de3" />

---

<img width="386" height="303" alt="Screenshot 2026-04-02 154014" src="https://github.com/user-attachments/assets/00ba138f-7f7e-45fc-b7dc-75e19d2274c8" />

---

### Task 2: Functions with Return Values
1. Create `disk_check.sh` with:
   - A function `check_disk` that checks disk usage of `/` using `df -h`
   - A function `check_memory` that checks free memory using `free -h`
   - A main section that calls both and prints the results

---

My answer:

<img width="1920" height="374" alt="image" src="https://github.com/user-attachments/assets/2c79b44f-1e41-4fd3-bd7e-e99899ed7db4" />

---

<img width="1210" height="804" alt="image" src="https://github.com/user-attachments/assets/79beb93c-ec2e-4b9e-9066-e5547ad012ae" />


---

### Task 3: Strict Mode — `set -euo pipefail`
1. Create `strict_demo.sh` with `set -euo pipefail` at the top
2. Try using an **undefined variable** — what happens with `set -u`?
3. Try a command that **fails** — what happens with `set -e`?
4. Try a **piped command** where one part fails — what happens with `set -o pipefail`?

**Document:** What does each flag do?
- `set -e` →
- `set -u` →
- `set -o pipefail` →

---

My answer:

<img width="1490" height="690" alt="image" src="https://github.com/user-attachments/assets/03a84d6b-fdbb-40e4-b890-bc470cbe2d2f" />

---

- set -e          --> exit if any command fails
- set -u          --> error on undefined variable
- set -o pipefail --> fails/error if any command fails in pipeline ( | ).

---

### Task 4: Local Variables
1. Create `local_demo.sh` with:
   - A function that uses `local` keyword for variables
   - Show that `local` variables don't leak outside the function
   - Compare with a function that uses regular variables

---

my answer:

<img width="1258" height="192" alt="image" src="https://github.com/user-attachments/assets/7a88ea7f-efd4-4877-ab9a-db7e68df81cd" />

---

<img width="1288" height="860" alt="image" src="https://github.com/user-attachments/assets/7266a9a2-bf1e-498e-82e1-4b4d6fc99ae5" />

---

### Task 5: Build a Script — System Info Reporter
Create `system_info.sh` that uses functions for everything:
1. A function to print **hostname and OS info**
2. A function to print **uptime**
3. A function to print **disk usage** (top 5 by size)
4. A function to print **memory usage**
5. A function to print **top 5 CPU-consuming processes**
6. A `main` function that calls all of the above with section headers
7. Use `set -euo pipefail` at the top

Output should look clean and readable.

---

My answer:

<img width="1800" height="856" alt="image" src="https://github.com/user-attachments/assets/4b0f95b1-7a9a-482c-a14f-eef9e9b9fe1e" />

---

<img width="1248" height="1522" alt="image" src="https://github.com/user-attachments/assets/e66d2221-55e8-4aa7-9842-52308b2c9971" />


---

## Scripts Created
      functions.sh  
      disk_check.sh  
      strict_demo.sh  
      local_demo.sh  
      system_info.sh  

## What I Learned
- How to use functions for clean scripts  
- Importance of strict mode (set -euo pipefail)  
- How to structure scripts.


Ciao Adios :)
