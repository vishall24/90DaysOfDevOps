# Day 46 – Reusable Workflows & Composite Actions

## Task
You've been writing workflows from scratch every time. In the real world, teams **don't repeat themselves** — they create reusable workflows that any repo can call like a function. Today you learn `workflow_call` and composite actions.

---

## Challenge Tasks

### Task 1: Understand `workflow_call`
Before writing any code, research and answer in your notes:
1. What is a **reusable workflow**?
2. What is the `workflow_call` trigger?
3. How is calling a reusable workflow different from using a regular action (`uses:`)?
4. Where must a reusable workflow file live?

---

## 1. Reusable Workflow

    A workflow that can be called by another workflow (like a function).

## 2. workflow_call

    A trigger that allows a workflow to be invoked by another workflow.

## 3. Difference from normal uses:

    Reusable workflow → calls entire workflow (jobs + steps)
    Action → used inside a step only

## 4. Location

    Must be inside:
    
    .github/workflows/


---

### Task 2: Create Your First Reusable Workflow
Create `.github/workflows/reusable-build.yml`:
1. Set the trigger to `workflow_call`
2. Add an `inputs:` section with:
   - `app_name` (string, required)
   - `environment` (string, required, default: `staging`)
3. Add a `secrets:` section with:
   - `docker_token` (required)
4. Create a job that:
   - Checks out the code
   - Prints `Building <app_name> for <environment>`
   - Prints `Docker token is set: true` (never print the actual secret)

**Verify:** This file alone won't run — it needs a caller. That's next.

---


---

### Task 3: Create a Caller Workflow
Create `.github/workflows/call-build.yml`:
1. Trigger on push to `main`
2. Add a job that uses your reusable workflow:
   ```yaml
   jobs:
     build:
       uses: ./.github/workflows/reusable-build.yml
       with:
         app_name: "my-web-app"
         environment: "production"
       secrets:
         docker_token: ${{ secrets.DOCKER_TOKEN }}
   ```
3. Push to `main` and watch it run

**Verify:** In the Actions tab, do you see the caller triggering the reusable workflow? Click into the job — can you see the inputs printed?

---

call build workflow: , and reusable workflow steps is executing:

<img width="1265" height="924" alt="image" src="https://github.com/user-attachments/assets/0dbbd8ed-95dd-4371-b126-b01521904e02" />


---

### Task 4: Add Outputs to the Reusable Workflow
Extend `reusable-build.yml`:
1. Add an `outputs:` section that exposes a `build_version` value
2. Inside the job, generate a version string (e.g., `v1.0-<short-sha>`) and set it as output
3. In your caller workflow, add a second job that:
   - Depends on the build job (`needs:`)
   - Reads and prints the `build_version` output

**Verify:** Does the second job print the version from the reusable workflow?

---

<img width="964" height="475" alt="image" src="https://github.com/user-attachments/assets/2ec90691-9ccc-480a-9fc1-a5694c5d31d5" />

---

### Task 5: Create a Composite Action
Create a **custom composite action** in your repo at `.github/actions/setup-and-greet/action.yml`:
1. Define inputs: `name` and `language` (default: `en`)
2. Add steps that:
   - Print a greeting in the specified language
   - Print the current date and runner OS
   - Set an output called `greeted` with value `true`
3. Use the composite action in a new workflow with `uses: ./.github/actions/setup-and-greet`

**Verify:** Does your custom action run and print the greeting?

---

<img width="1390" height="721" alt="image" src="https://github.com/user-attachments/assets/7ed93378-6ee8-41ec-ae27-b29d36e29200" />

---

### Task 6: Reusable Workflow vs Composite Action
Fill this in your notes:

| | Reusable Workflow | Composite Action |
|---|---|---|
| Triggered by | `workflow_call` | `uses:` in a step |
| Can contain jobs? | yes | No |
| Can contain multiple steps? | yes | yes |
| Lives where? | .github/workflows/ | .github/actions/ |
| Can accept secrets directly? | yes | No(indirect via env) |
| Best for | full pipeline | reusable logic |

---


## action.yaml: location ( .github/actions/setup-and-greet/ )

    name: Setup and Greet
    
    inputs:
      name:
        required: true
      language:
        required: false
        default: en
      outputs:
        greeted:
        value: "true"
    
    runs:
      using: "composite"  
    
      steps:
        - name: Greet
          run: |
            if [ "${{ inputs.language }}" == "en" ]; then
              echo "Hello ${{ inputs.name }}"
            else
              echo "Hi ${{ inputs.name }}"
            fi
          shell: bash
    
        - name: Print system info
          run: | 
            date 
            uname -a
          shell: bash


## call-build.yml:

    name: Call Reusable Workflow
    
    on:
      push:
        branches: [main]
    
    jobs:
        call-build:
            uses: ./.github/workflows/reusable-build.yml
            with:
                app_name: "my-web-app"
                environment: "dev"
            secrets:
                docker_token: ${{ secrets.DOCKER_TOKEN }}
    
        print-version:
            runs-on: ubuntu-latest
            needs: call-build
    
            steps:
              - run: echo "Version is ${{ needs.call-build.outputs.build_version }}"


## reusable-build:

    name: Reusable Build
    
    on:
      workflow_call:
        inputs:
          app_name:
            required: true
            type: string
            default: write your app name
          environment:
            required: true
            type: string
            default: staging
        secrets:
          docker_token:
            required: true
        outputs:
            build_version:
                value: ${{ jobs.build.outputs.version }}
    
    
    jobs:
      build:
        runs-on: ubuntu-latest
        outputs: 
            version: ${{ steps.set_version.outputs.version }}
    
        environment: ${{ inputs.environment }}
    
        steps:
          - uses: actions/checkout@v4
    
          - name: print build info
            run: |
              echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"
    
          - name: check secret
            run: |
              if [ -n "${{ secrets.docker_token }}" ]; then
                echo "Docker token is set: true"
              fi
    
          - id: set_version
            run: echo "version=v1.0-${GITHUB_SHA::7}" >> $GITHUB_OUTPUT
    
      test-actions:
        runs-on: ubuntu-latest
        
        steps:
            
            - uses: actions/checkout@v4
    
            - name: Use custom action
              uses: ./.github/actions/setup-and-greet
              with:
                name: Vishal
                language: en
    
