# Day 38 – YAML Basics

## Task
Before writing a single CI/CD pipeline, you need to get comfortable with **YAML** — the language every pipeline is written in.

You will:
- Understand YAML syntax and rules
- Write YAML files by hand
- Validate them

---

## Challenge Tasks

### Task 1: Key-Value Pairs
Create `person.yaml` that describes yourself with:
- `name`
- `role`
- `experience_years`
- `learning` (a boolean)

**Verify:** Run `cat person.yaml` — does it look clean? No tabs?

---

<img width="980" height="230" alt="image" src="https://github.com/user-attachments/assets/327f23e6-e401-4596-9f6e-91279f9b3a69" />

Rules:

    key: value
    No tabs, only spaces
    true is boolean (not "true")

---

### Task 2: Lists
Add to `person.yaml`:
- `tools` — a list of 5 DevOps tools you know or are learning
- `hobbies` — a list using the inline format `[item1, item2]`

Write in your notes: What are the two ways to write a list in YAML?

---

<img width="776" height="822" alt="image" src="https://github.com/user-attachments/assets/a9812f3c-f2b2-47bd-8304-9b533934e45e" />

2 types of lists:

    Block list:
      - item1
      - item2
    
    Inline list: [item1, item2]

---

### Task 3: Nested Objects
Create `server.yaml` that describes a server:
- `server` with nested keys: `name`, `ip`, `port`
- `database` with nested keys: `host`, `name`, `credentials` (nested further: `user`, `password`)

**Verify:** Try adding a tab instead of spaces — what happens when you validate it?

---


<img width="498" height="482" alt="image" src="https://github.com/user-attachments/assets/baba8d51-b52d-44e7-b605-cc92f57fdfdb" />

Important:

Indentation = structure
2 spaces standard


Tried wrong indentation:

    server:
    	name: wrong

   YAML will break
   YAML = NO tabs allowed

---

### Task 4: Multi-line Strings
In `server.yaml`, add a `startup_script` field using:
1. The `|` block style (preserves newlines)
2. The `>` fold style (folds into one line)

Write in your notes: When would you use `|` vs `>`?

---

<img width="506" height="242" alt="image" src="https://github.com/user-attachments/assets/c7442915-71f3-4caa-9670-0b7194a8690b" />

Using | (Literal block)

    startup_script: |
      echo "Starting app"
      npm install
      npm start

 This becomes:

    echo "Starting app"
    npm install
    npm start

Lines stay EXACTLY same

 
Perfect for:

    bash scripts
    configs
    commands

    
Using > (Folded block)

    startup_script_folded: >
      echo "Starting app"
      npm install
      npm start

This becomes:

    echo "Starting app" npm install npm start

All lines become ONE line

Why this happens, > replaces newline (\n) with space, | keeps newline

---

### Task 5: Validate Your YAML
1. Install `yamllint` or use an online validator
2. Validate both your YAML files
3. Intentionally break the indentation — what error do you get?
4. Fix it and validate again

---
Errors I got:

<img width="1522" height="402" alt="image" src="https://github.com/user-attachments/assets/364b8bc6-6a55-4a47-b1b9-9e9858b13ea4" />

Fixed it and validated and errors got resolved.

<img width="772" height="908" alt="image" src="https://github.com/user-attachments/assets/bba27156-8186-4f1d-a93d-f3f6302fb3bc" />

### Task 6: Spot the Difference
Read both blocks and write what's wrong with the second one:

```yaml
# Block 1 - correct
name: devops
tools:
  - docker
  - kubernetes
```

```yaml
# Block 2 - broken
name: devops
tools:
- docker
  - kubernetes
```

---

the - docker has bad indentation it should have 2 spaces before the dash - same as - kubernetes.


## person.yaml
    
    ---
    
    # key-value pairs
    
    name: Vishal
    role: DevOps engineer
    experience_years: 3
    learning: true
    
    # lists
    
    tools:
      - docker
      - kubernetes
      - git
      - linux
      - github-actions
    
    # List using inline format
    hobbies: [gaming, gym, editing]

## server.yaml
 
    ---
    
    server:
      name: my-server
      ip: 192.168.1.1
      port: 8080
    
    database:
      host: localhost
      name: mydb
      credentials:
        user: admin
        password: admin
    
    startup_script: |
      echo "starting app"
      npm install
      npm start

## What I Learned

- YAML uses spaces, not tabs
- Indentation defines structure
- Lists can be block or inline
- | keeps format, > converts to single line
