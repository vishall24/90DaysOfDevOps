# Day 22 – Introduction to Git: Your First Repository

## Task

Today marks the beginning of your Git journey. Git is the backbone of modern DevOps — every tool, pipeline, and workflow revolves around version control. Before diving into advanced concepts, you need to get comfortable with the basics by doing.

You will:
- Understand what Git is and why it matters
- Set up your first Git repository from scratch
- Start building a living document of Git commands

---

## Challenge Tasks

### Task 1: Install and Configure Git
1. Verify Git is installed on your machine
2. Set up your Git identity — name and email
3. Verify your configuration

---

### Task 2: Create Your Git Project
1. Create a new folder called `devops-git-practice`
2. Initialize it as a Git repository
3. Check the status — read and understand what Git is telling you
4. Explore the hidden `.git/` directory — look at what's inside

---

### Task 3: Create Your Git Commands Reference
1. Create a file called `git-commands.md` inside the repo
2. Add the Git commands you've used so far, organized by category:
   - **Setup & Config**
   - **Basic Workflow**
   - **Viewing Changes**
3. For each command, write:
   - What it does (1 line)
   - An example of how to use it

---

### Task 4: Stage and Commit
1. Stage your file
2. Check what's staged
3. Commit with a meaningful message
4. View your commit history

---

### Task 5: Make More Changes and Build History
1. Edit `git-commands.md` — add more commands as you discover them
2. Check what changed since your last commit
3. Stage and commit again with a different, descriptive message
4. Repeat this process at least **3 times** so you have multiple commits in your history
5. View the full history in a compact format

---

### Task 6: Understand the Git Workflow
Answer these questions in your own words (add them to a `day-22-notes.md` file):
1. What is the difference between `git add` and `git commit`?
2. What does the **staging area** do? Why doesn't Git just commit directly?
3. What information does `git log` show you?
4. What is the `.git/` folder and what happens if you delete it?
5. What is the difference between a **working directory**, **staging area**, and **repository**?

---

My answer:

## 1. git add vs git commit
    git add -> moves changes to staging  
    git commit -> saves the staging changes  

## 2. Staging Area
    It is a buffer where changes are prepared before commit. (git add .)

## 3. git log
    Shows commit history with messages, author, time (git log)

## 4. .git folder
    Contains all repo data. If deleted → history lost (kinda metadata)

## 5. Working vs Staging vs Repository
    Working → current files  
    Staging → ready to commit  (git add .)
    Repository → saved history 

ScreenShot of git log --oneline::

<img width="1568" height="338" alt="image" src="https://github.com/user-attachments/assets/2f872e0d-f5a0-4ff2-a3e1-d57d823b7c67" />

checked all logs by going inside .git folder ::

<img width="2862" height="1032" alt="image" src="https://github.com/user-attachments/assets/2acd0357-494f-42c4-847c-939e801a41f9" />



Ciaos Adios  ;)
