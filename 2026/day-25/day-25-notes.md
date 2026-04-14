# Day 25 – Git Reset vs Revert & Branching Strategies

## Task

You'll learn how to **undo mistakes** safely — one of the most important skills in Git. You'll also explore **branching strategies** used by real engineering teams to manage code at scale.

---

## Challenge Tasks

### Task 1: Git Reset — Hands-On
1. Make 3 commits in your practice repo (commit A, B, C)
2. Use `git reset --soft` to go back one commit — what happens to the changes?
3. Re-commit, then use `git reset --mixed` to go back one commit — what happens now?
4. Re-commit, then use `git reset --hard` to go back one commit — what happens this time?
5. Answer in your notes:
   - What is the difference between `--soft`, `--mixed`, and `--hard`?
   - Which one is destructive and why?
   - When would you use each one?
   - Should you ever use `git reset` on commits that are already pushed?

---

My answer:

<img width="857" height="841" alt="image" src="https://github.com/user-attachments/assets/006f037e-bbed-43a9-b1bd-6984855d87db" />

- Observed the commit ID got removed and came back to one commit back , But the changes were as it is no changes in local.

- After performing **git reset --mixed HEAD~1**, Observed that the commit ID went one step back, but the changes were present
  in local, so whats the difference between --sotf & --mixed ? :: in --mixed the changes were not in working dir(unstaged)
  But in --sotf the changes were staged (git add . )

- <img width="862" height="891" alt="image" src="https://github.com/user-attachments/assets/662a0785-ce97-41df-8e07-9a92548b06fd" />
  Here using --hard , the commit ID as well as the local changes got removed

- What is the difference between `--soft`, `--mixed`, and `--hard`?
  == --sotf  : will just removes the commit ID but the changes remains staged, changes will be in dir.
     --mixed : will removes the commit ID but the changes gets moved to unstaged, changes will be in dir.
     --hard  : will removes the commit ID as well as the local changes all gone.


---

### Task 2: Git Revert — Hands-On
1. Make 3 commits (commit X, Y, Z)
2. Revert commit Y (the middle one) — what happens?
3. Check `git log` — is commit Y still in the history?
4. Answer in your notes:
   - How is `git revert` different from `git reset`?
   - Why is revert considered **safer** than reset for shared branches?
   - When would you use revert vs reset?

---

My answer:

<img width="870" height="257" alt="image" src="https://github.com/user-attachments/assets/887a1364-eadc-49c2-af84-691733921e49" />

Reset vs Revert:

reset -> deletes history
revert -> adds new undo commit

safe:

revert is safe since it preserves the history and only use the reset when you are 100% sure.

---

### Task 3: Reset vs Revert — Summary
Create a comparison in your notes:

| | `git reset` | `git revert` |
|---|---|---|
| What it does | moves HEAD~1 one commit back | revert the changes and creates new commit |
| Removes commit from history? | Yes | No |
| Safe for shared/pushed branches? | No | Yes |
| When to use | local fixes | Shared repo fixes |

---

### Task 4: Branching Strategies
Research the following branching strategies and document each in your notes with:
- How it works (short description)
- A simple diagram or flow (text-based is fine)
- When/where it's used
- Pros and cons

1. **GitFlow** — develop, feature, release, hotfix branches
2. **GitHub Flow** — simple, single main branch + feature branches
3. **Trunk-Based Development** — everyone commits to main, short-lived branches
4. Answer:
   - Which strategy would you use for a startup shipping fast?
   - Which strategy would you use for a large team with scheduled releases?
   - Which one does your favorite open-source project use? (check any repo on GitHub)

---

My answer:

 1. GitFlow

 Flow:

    main → production  
    develop → working  
    feature → new work  
    release → prepare release  
    hotfix → urgent fix  

 Use:

    big teams
    scheduled releases
   
 2. GitHub Flow

 Flow:

    main → always deployable  
    feature branch → PR → merge  

 Use:

    startups
    fast deployment

 3. Trunk-Based

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

---


Ciao Adios :)
