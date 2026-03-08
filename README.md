
# Day 43 – Jobs, Steps, Environment Variables & Conditionals

## Overview

Today I learned how to control the flow of a GitHub Actions pipeline.
A workflow can contain multiple jobs, and each job contains multiple steps.
Using job dependencies, environment variables, outputs, and conditions, we can control how and when different parts of the pipeline run.

These concepts are important because real CI/CD pipelines must coordinate many tasks such as building code, running tests, and deploying applications.

---

# 1. Multi-Job Workflow

A workflow can contain multiple jobs.
Each job runs on its own runner (virtual machine).

Example workflow:

```
build → test → deploy
```

### Workflow Example

```yaml
name: Multi Job Pipeline

on: push

jobs:

  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build step
        run: echo "Building the app"

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Test step
        run: echo "Running tests"

  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Deploy step
        run: echo "Deploying"
```

### Explanation

**build job**

Runs first and simulates building the application.

```
echo "Building the app"
```

**test job**

Uses:

```
needs: build
```

This means the test job will run **only after the build job succeeds**.

**deploy job**

Uses:

```
needs: test
```

This means deployment will only run after testing finishes successfully.

### Pipeline Flow

```
build
   ↓
test
   ↓
deploy
```

This dependency chain ensures that the application is not deployed unless the build and tests succeed.

---

# 2. Environment Variables

Environment variables store values that can be reused across the workflow.

They can be defined at three levels:

1. Workflow level
2. Job level
3. Step level

Example workflow:

```yaml
name: Environment Variables Demo

on: push

env:
  APP_NAME: myapp

jobs:
  show-env:
    runs-on: ubuntu-latest

    env:
      ENVIRONMENT: staging

    steps:
      - name: Print environment variables
        env:
          VERSION: 1.0.0
        run: |
          echo "App Name: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $VERSION"

      - name: GitHub context variables
        run: |
          echo "Commit SHA: ${{ github.sha }}"
          echo "Triggered by: ${{ github.actor }}"
```

### Variable Levels

**Workflow Level**

```
env:
  APP_NAME: myapp
```

Accessible in all jobs and steps.

**Job Level**

```
ENVIRONMENT: staging
```

Accessible only within that specific job.

**Step Level**

```
VERSION: 1.0.0
```

Accessible only in that step.

### GitHub Context Variables

GitHub automatically provides useful variables.

Examples:

```
github.sha
github.actor
```

Meaning:

* **github.sha** → commit ID that triggered the workflow
* **github.actor** → the user who triggered the workflow

---

# 3. Job Outputs

Jobs run on separate virtual machines, so they cannot directly share data.
To pass information between jobs we use **job outputs**.

### Example

First job generates a value (today's date).
Second job reads and prints that value.

```yaml
name: Job Output Example

on: push

jobs:

  generate-date:
    runs-on: ubuntu-latest

    outputs:
      today: ${{ steps.date_step.outputs.today }}

    steps:
      - id: date_step
        run: echo "today=$(date)" >> $GITHUB_OUTPUT

  print-date:
    runs-on: ubuntu-latest
    needs: generate-date

    steps:
      - name: Print date
        run: echo "Date from previous job: ${{ needs.generate-date.outputs.today }}"
```

### Explanation

**Step Output**

```
echo "today=$(date)" >> $GITHUB_OUTPUT
```

Creates an output variable called `today`.

**Job Output**

```
outputs:
  today: ${{ steps.date_step.outputs.today }}
```

Exposes the step output as a job output.

**Accessing Output**

```
${{ needs.generate-date.outputs.today }}
```

This retrieves the value from the previous job.

### Why Outputs Are Useful

In real pipelines outputs are used to pass information like:

* Docker image tag
* Build version
* Artifact path
* Deployment environment

Example pipeline:

```
Build Job → generates image tag
Deploy Job → deploys that tag
```

---

# 4. Conditionals

Conditionals control when jobs or steps should run.

Example workflow:

```yaml
name: Conditional Workflow

on:
  push:
  pull_request:

jobs:
  example:
    runs-on: ubuntu-latest

    steps:
      - name: Run only on main branch
        if: github.ref == 'refs/heads/main'
        run: echo "This runs only on main branch"

      - name: Failing step
        run: exit 1
        continue-on-error: true

      - name: Run if previous step failed
        if: failure()
        run: echo "Previous step failed"

  push-only-job:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - run: echo "Runs only on push events"
```

### Explanation

**Run only on main branch**

```
if: github.ref == 'refs/heads/main'
```

This step runs only when the workflow is triggered from the main branch.

---

**Run step if previous step failed**

```
if: failure()
```

This is useful for sending alerts or rollback operations.

---

**Run job only on push**

```
if: github.event_name == 'push'
```

Prevents the job from running on pull requests.

---

**continue-on-error**

```
continue-on-error: true
```

If this step fails, the workflow will continue instead of stopping.

---

# 5. Smart Pipeline Example

This workflow combines everything learned.

```yaml
name: Smart Pipeline

on:
  push:

jobs:

  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running lint checks"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running tests"

  summary:
    runs-on: ubuntu-latest
    needs: [lint, test]

    steps:
      - name: Print branch info
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "Main branch push"
          else
            echo "Feature branch push"
          fi

      - name: Print commit message
        run: echo "Commit message: ${{ github.event.head_commit.message }}"
```

### Pipeline Execution

```
        push
          │
    ┌─────┴─────┐
    │           │
  lint        test
    │           │
    └─────┬─────┘
          │
       summary
```

**lint and test jobs run in parallel.**

The **summary job waits for both jobs to finish** using:

```
needs: [lint, test]
```

The summary job prints:

* Whether the push was on the main branch or a feature branch
* The commit message that triggered the pipeline

---

# Key Concepts Learned

* Workflows can contain multiple jobs.
* Jobs contain steps.
* `needs:` controls job dependencies.
* Environment variables allow sharing configuration values.
* Outputs allow passing data between jobs.
* Conditionals allow pipelines to run only when certain conditions are met.

---

# Conclusion

Today I learned how to design a smarter CI/CD pipeline by controlling job order, sharing data between jobs, and running steps conditionally.
These features are essential for building production-grade DevOps pipelines.
