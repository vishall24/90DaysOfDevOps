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





