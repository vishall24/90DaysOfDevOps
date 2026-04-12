# Day 23 – Git Branching & Working with GitHub

## Task

Now that you know how to create repos, stage, and commit — it's time to learn the most powerful concept in Git: **branching**. Branches let you work on features, fixes, and experiments in isolation without breaking your main code. You'll also push your work to GitHub for the first time.

---

## Challenge Tasks

---

### Task 1: Understanding Branches

== 

- Observed there was fast-forward -> Happens when no new commits on target branch so the merge just forward with the merge
  main has no new commits
  Git just moves the pointer forward

- Git merge commit :
  Both branches have new commits
  Histories have diverged
  
- Observed CONFLICT message after editing the same line in two different branches and running git merge feature-signup,
  resolved and closed the merge.

---

### Task 2: Git Rebase — Hands-On

- rebase created conflict while rebasing 
- Observed all the commmits from the master branch came into feature-dashboard branch after rebasing successful.
- History for merge is non - linear and for rebase it is linear and clear.
- Why should you **never rebase commits that have been pushed and shared** with others?
  :
   Because rebase
    - rewrites commit history
    - changes commit IDs
    
     If already pushed:
    - others’ history breaks
    - causes conflicts + confusion
    - Steps to resolve the rebase conflict

  conflict
  fix file
  git add
  git rebase --continue

---

### Task 3: Squash Commit vs Merge Commit

- Observed that there was only one commit for a lot of commits history. , Multiple Commits --> One commit history
- difference between merge and --squash was there was one single commit for all commits where as in merge there was all commit history.
- the is trade-off squash? you will lose individual commit history and wont be able to identify who did what?

---

### Task 4: Git Stash — Hands-On

  Difference: stash pop vs stash apply

   git stash pop:
  
    applies stash 
    removes it from stash list 

   git stash apply:
    
    applies stash 
    keeps it in stash list 
    When to use stash in real world?
    
   Use stash when:
    
    you have unfinished work
    need to switch branches urgently
    don’t want to commit incomplete code

---

### Task 5: Cherry Picking

  What does cherry-pick do?

  -  Cherry-pick applies a specific commit from another branch to current branch

  Instead of merging full branch -> pick only needed change

  When to use cherry-pick?

  - Use when:
    need specific fix (hotfix)
    don’t want full feature branch
    urgent production fix


Ciao Adios :)
