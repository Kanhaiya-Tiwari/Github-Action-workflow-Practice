# Day 44 – Secrets, Artifacts & Running Real Tests in CI

## Overview

In this exercise, the CI pipeline begins performing real tasks instead of simple echo commands.

Today I learned how to:

* Store sensitive information securely using **GitHub Secrets**
* Use secrets safely in workflows
* Save build outputs using **artifacts**
* Share files between jobs
* Run real scripts/tests inside CI
* Improve pipeline speed using **caching**

These features are essential for real-world CI/CD pipelines.

# 1. GitHub Secrets

CI pipelines often require sensitive data such as:

* API keys
* Docker credentials
* database passwords
* cloud access tokens

Hardcoding these values inside a workflow is dangerous because anyone with repository access could see them.

GitHub solves this problem using **Secrets**.

Secrets are encrypted values stored in the repository settings and accessed securely inside workflows.

---

## Creating a Secret

Steps:

1. Open your GitHub repository
2. Go to **Settings**
3. Click **Secrets and Variables**
4. Select **Actions**
5. Click **New Repository Secret**

Create a secret:

```
Name: MY_SECRET_MESSAGE
Value: hello-secret
```

---

## Workflow Example

```yaml
name: Secret Example

on: push

jobs:
  secret-check:
    runs-on: ubuntu-latest

    steps:
      - name: Check if secret exists
        run: |
          if [ -n "${{ secrets.MY_SECRET_MESSAGE }}" ]; then
            echo "The secret is set: true"
          else
            echo "The secret is missing"
          fi
```

### Explanation

Secret values are accessed using:

```
${{ secrets.SECRET_NAME }}
```

Example:

```
${{ secrets.MY_SECRET_MESSAGE }}
```

The workflow checks whether the secret exists without printing its value.

---

## What Happens if You Print a Secret?

Example:

```yaml
run: echo ${{ secrets.MY_SECRET_MESSAGE }}
```

GitHub automatically masks secrets in logs.

Output example:

```
***
```

This prevents secrets from being exposed in CI logs.

---

## Why You Should Never Print Secrets

Secrets should never appear in logs because:

* Logs can be viewed by collaborators
* Logs may be stored permanently
* Attackers could steal credentials

Therefore secrets should only be used internally by commands and never displayed.

---

# 2. Using Secrets as Environment Variables

Secrets can be passed into steps as environment variables.

Example:

```yaml
name: Secret Environment Variable

on: push

jobs:
  secret-env:
    runs-on: ubuntu-latest

    steps:
      - name: Use secret as environment variable
        env:
          SECRET_MESSAGE: ${{ secrets.MY_SECRET_MESSAGE }}
        run: |
          echo "Secret is configured and available"
```

The secret is available to the shell but not printed.

---

## Docker Credentials

Create two secrets for future workflows:

```
DOCKER_USERNAME
DOCKER_TOKEN
```

These will later be used for:

* logging into Docker Hub
* pushing container images

Example usage:

```yaml
env:
  DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
  DOCKER_TOKEN: ${{ secrets.DOCKER_TOKEN }}
```

---

# 3. Uploading Artifacts

Artifacts are files generated during a workflow that you want to keep after the workflow finishes.

Examples of artifacts:

* test reports
* build logs
* compiled binaries
* coverage reports

Artifacts allow you to download these files from the GitHub Actions interface.

---

## Example Workflow

```yaml
name: Artifact Upload Example

on: push

jobs:
  create-report:
    runs-on: ubuntu-latest

    steps:
      - name: Generate report
        run: |
          echo "Test Report" > report.txt
          echo "All tests passed" >> report.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: report.txt
```

### Explanation

This step creates a file:

```
report.txt
```

Then the artifact action uploads it:

```
actions/upload-artifact@v4
```

Artifact details:

```
name → test-report
path → report.txt
```

---

## Downloading Artifact from GitHub

After the workflow finishes:

1. Open **Actions tab**
2. Select the workflow run
3. Scroll to **Artifacts**
4. Download the file

This allows you to inspect outputs from the pipeline.

---

# 4. Download Artifacts Between Jobs

Artifacts can also be used to transfer files between jobs.

Example pipeline:

```
Job 1 → build file
Job 2 → use file
```

---

## Workflow Example

```yaml
name: Artifact Sharing Example

on: push

jobs:

  generate-file:
    runs-on: ubuntu-latest

    steps:
      - name: Create file
        run: echo "Hello from Job 1" > message.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: shared-file
          path: message.txt


  read-file:
    runs-on: ubuntu-latest
    needs: generate-file

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: shared-file

      - name: Print file
        run: cat message.txt
```

### Execution Flow

```
generate-file
      ↓
upload artifact
      ↓
read-file
      ↓
download artifact
      ↓
print contents
```

---

## When Artifacts Are Used in Real Pipelines

Artifacts are commonly used for:

* sharing compiled builds
* storing test reports
* passing deployment packages
* storing logs for debugging

Example:

```
Build Job → generate Docker image
Test Job → download build
Deploy Job → deploy build
```

---

# 5. Running Real Tests in CI

A CI pipeline should verify code automatically.

Example:

```
run script
if script fails → pipeline fails
```

---

## Example Script

Create a script inside your repo.

Example:

```
scripts/test.sh
```

```bash
#!/bin/bash
echo "Running tests..."

exit 0
```

---

## Workflow to Run Script

```yaml
name: Run Tests

on: push

jobs:
  test-script:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run test script
        run: |
          chmod +x scripts/test.sh
          ./scripts/test.sh
```

Explanation:

```
actions/checkout
```

Downloads the repository code to the runner.

Then the script is executed.

---

## Testing Pipeline Failure

Modify the script:

```
exit 1
```

This causes the pipeline to fail.

GitHub will mark the workflow as:

```
FAILED (red)
```

After fixing the script:

```
exit 0
```

The pipeline becomes:

```
SUCCESS (green)
```

---

# 6. Caching Dependencies

Installing dependencies repeatedly slows CI pipelines.

Caching stores dependencies so future runs are faster.

Example dependencies:

* npm packages
* Python libraries
* Maven dependencies

---

## Cache Example

```yaml
name: Dependency Cache

on: push

jobs:
  cache-demo:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: npm-cache-${{ hashFiles('package-lock.json') }}

      - name: Install dependencies
        run: npm install
```

---

## How Caching Works

First run:

```
No cache found
Install dependencies
Save cache
```

Second run:

```
Cache restored
Skip downloading packages
Pipeline runs faster
```

---

## Where Cache Is Stored

GitHub stores the cache in its own infrastructure.

The cache is linked to:

```
repository
workflow
cache key
```

---

# Key Concepts Learned

Today I learned several critical CI/CD concepts:

* **Secrets** securely store sensitive credentials.
* Secrets are accessed using `${{ secrets.SECRET_NAME }}`.
* **Artifacts** store files generated during workflows.
* Artifacts can also transfer files between jobs.
* CI pipelines can run real scripts and detect failures.
* **Caching** improves pipeline speed by storing dependencies.

---

# Conclusion

Day 44 introduced important CI/CD features used in real DevOps pipelines.

Secrets ensure secure handling of credentials, artifacts allow build outputs to be stored and shared, and caching improves performance. Running real tests ensures code reliability before deployment.
