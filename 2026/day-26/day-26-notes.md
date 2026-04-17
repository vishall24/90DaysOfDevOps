# Day 26 – GitHub CLI: Manage GitHub from Your Terminal

## Task

Every time you switch to the browser to create a PR, check an issue, or manage a repo — you lose context. The **GitHub CLI (`gh`)** lets you do all of that without leaving your terminal. For DevOps engineers, this is essential — especially when you start automating workflows, scripting PR reviews, and managing repos at scale.

---

## Challenge Tasks

### Task 1: Install and Authenticate
1. Install the GitHub CLI on your machine
2. Authenticate with your GitHub account
3. Verify you're logged in and check which account is active
4. Answer in your notes: What authentication methods does `gh` support?

---

My answer:

gh supports these authentication methods:

    GitHub.com
    HTTPS
    Login via browser

---

### Task 2: Working with Repositories
1. Create a **new GitHub repo** directly from the terminal — make it public with a README
2. Clone a repo using `gh` instead of `git clone`
3. View details of one of your repos from the terminal
4. List all your repositories
5. Open a repo in your browser directly from the terminal
6. Delete the test repo you created (be careful!)

---

My answer:

<img width="997" height="130" alt="image" src="https://github.com/user-attachments/assets/b06e2dcb-d0ea-436e-be96-997882b03b24" />

created and cloned the repo named "test-gh-repo"

<img width="1791" height="826" alt="image" src="https://github.com/user-attachments/assets/e07cacb6-3246-42b4-88cf-9ac23daff23c" />

<img width="1125" height="205" alt="image" src="https://github.com/user-attachments/assets/d1335c69-d805-4eea-8651-6d997afe696f" />

<img width="1393" height="142" alt="image" src="https://github.com/user-attachments/assets/2a5510c6-348b-4b3a-a02d-4795e62b406d" />

<img width="1616" height="625" alt="image" src="https://github.com/user-attachments/assets/8e63bb9f-22b9-4423-99f2-6bd878cb7343" />

---

### Task 3: Issues
1. Create an issue on one of your repos from the terminal — give it a title, body, and a label
2. List all open issues on that repo
3. View a specific issue by its number
4. Close an issue from the terminal
5. Answer in your notes: How could you use `gh issue` in a script or automation?

---
My answer:

<img width="1393" height="142" alt="image" src="https://github.com/user-attachments/assets/c433cebf-a59a-455d-975c-41b6452cdeda" />

<img width="1877" height="288" alt="image" src="https://github.com/user-attachments/assets/5526282e-dea6-4011-8170-f9d5cec1f3d7" />

<img width="867" height="168" alt="image" src="https://github.com/user-attachments/assets/357fcdc6-e58e-4ce8-ae9c-9c1f8b137aa6" />

<img width="814" height="64" alt="image" src="https://github.com/user-attachments/assets/a7a340ad-97f3-4a63-b7bc-d7bee0e342fb" />

closed:

<img width="1918" height="310" alt="image" src="https://github.com/user-attachments/assets/70995098-5395-4cf1-8bae-6610024186c1" />

I can use gh issue when there is any requirement like a developer wants to automate the processes of creating and closing the 
issue.

---

### Task 4: Pull Requests
1. Create a branch, make a change, push it, and create a **pull request** entirely from the terminal
2. List all open PRs on a repo
3. View the details of your PR — check its status, reviewers, and checks
4. Merge your PR from the terminal
5. Answer in your notes:
   - What merge methods does `gh pr merge` support?
   - How would you review someone else's PR using `gh`?

---

<img width="1170" height="184" alt="image" src="https://github.com/user-attachments/assets/a82c2070-f98a-41eb-9b7e-672140ecd541" />

<img width="1009" height="199" alt="image" src="https://github.com/user-attachments/assets/8dd0ed5a-6a2f-4da2-8f66-72282f3b99c6" />

<img width="887" height="184" alt="image" src="https://github.com/user-attachments/assets/b0701da4-da61-4dc7-963d-9032b40b86bb" />

<img width="1146" height="169" alt="image" src="https://github.com/user-attachments/assets/bb2a01a4-0385-4cac-9062-c74ef02e8144" />

<img width="1533" height="342" alt="image" src="https://github.com/user-attachments/assets/82c9166c-e3d8-4066-833c-43232757025d" />

<img width="1073" height="624" alt="image" src="https://github.com/user-attachments/assets/9cb4af97-eb10-49c4-8f97-428c809f80c3" />

- gh pr merge supports -
    - squash & merge
    - merge
    - rebase

---

### Task 5: GitHub Actions & Workflows (Preview)
1. List the workflow runs on any public repo that uses GitHub Actions
2. View the status of a specific workflow run
3. Answer in your notes: How could `gh run` and `gh workflow` be useful in a CI/CD pipeline?

(Don't worry if you haven't learned GitHub Actions yet — this is a preview for upcoming days)

---

My answer:

    checking CI/CD status
    debugging pipeline

---

### Task 6: Useful `gh` Tricks
Explore and try these — add the ones you find useful to your `git-commands.md`:
1. `gh api` — make raw GitHub API calls from the terminal
2. `gh gist` — create and manage GitHub Gists
3. `gh release` — create and manage releases
4. `gh alias` — create shortcuts for commands you use often
5. `gh search repos` — search GitHub repos from the terminal

---

My answer:

    gh search repos devops
    gh gist create file.txt

---

## Authentication
gh supports browser login and token authentication

## Repo Management
Can create, view, list, and delete repos from terminal

## Issues
Can create, list, view, and close issues using CLI

## Pull Requests
Can create and merge PRs directly from terminal

## Merge Methods
merge, squash, rebase

## gh in CI/CD
Useful to automate PRs, check workflows, manage repos

