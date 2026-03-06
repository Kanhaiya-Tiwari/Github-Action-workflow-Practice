# Day 42 – Runners: GitHub-Hosted & Self-Hosted

Today I learned about **GitHub Actions runners**, which are machines that execute workflow jobs.
GitHub provides **hosted runners**, and developers can also configure **self-hosted runners** on their own machines or cloud servers.

Understanding runners is important because every CI/CD pipeline needs a machine to execute the workflow steps.

---

# Task 1 – GitHub-Hosted Runners

I created a workflow that runs three jobs on different operating systems.

Supported runners:

* ubuntu-latest
* windows-latest
* macos-latest

### Workflow File

```
.github/workflows/runners.yml
```

### YAML

```yaml
name: Runner OS Test

on: push

jobs:

  ubuntu-job:
    runs-on: ubuntu-latest
    steps:
      - name: Print system info
        run: |
          echo "Operating System:"
          uname -a
          echo "Hostname:"
          hostname
          echo "Current User:"
          whoami

  windows-job:
    runs-on: windows-latest
    steps:
      - name: Print system info
        run: |
          echo OS Info
          hostname
          whoami

  macos-job:
    runs-on: macos-latest
    steps:
      - name: Print system info
        run: |
          echo "Operating System:"
          uname -a
          echo "Hostname:"
          hostname
          echo "Current User:"
          whoami
```

### What is a GitHub-hosted runner?

A **GitHub-hosted runner** is a virtual machine provided and managed by GitHub that executes workflow jobs.

### Who manages it?

GitHub manages:

* Infrastructure
* Operating system updates
* Installed software
* Security patches
* Cleanup after job completion

Developers only need to define the workflow.

---

# Task 2 – Exploring Pre-installed Tools

GitHub runners come with many tools already installed.

I ran commands to check installed tools.

```yaml
jobs:
  check-tools:
    runs-on: ubuntu-latest

    steps:
      - name: Check installed tools
        run: |
          docker --version
          python --version
          node --version
          git --version
```

### Why pre-installed tools matter

Pre-installed tools reduce pipeline setup time.

Instead of installing tools every time, the runner already includes common development tools such as:

* Docker
* Python
* Node.js
* Git
* Java
* .NET

This makes CI/CD pipelines faster and easier to maintain.

---

# Task 3 – Setting Up a Self-Hosted Runner

I configured a **self-hosted runner** connected to my GitHub repository.

Steps:

1. Go to **Repository Settings**
2. Navigate to **Actions → Runners**
3. Click **New self-hosted runner**
4. Choose the operating system
5. Download the runner package
6. Configure the runner using the provided token
7. Start the runner

Example setup commands:

```bash
mkdir actions-runner
cd actions-runner

curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz

tar xzf actions-runner-linux-x64.tar.gz

./config.sh --url https://github.com/USERNAME/REPO --token TOKEN
```

Start the runner:

```bash
./run.sh
```

After starting the runner, GitHub shows it as **Idle** in the repository runners list.

---

# Task 4 – Running Jobs on Self-Hosted Runner

I created a workflow to run a job on my self-hosted runner.

### Workflow File

```
.github/workflows/self-hosted.yml
```

### YAML

```yaml
name: Self Hosted Runner Test

on: workflow_dispatch

jobs:
  self-hosted-job:
    runs-on: self-hosted

    steps:
      - name: Show hostname
        run: hostname

      - name: Show working directory
        run: pwd

      - name: Create test file
        run: echo "Hello from self-hosted runner" > runner-test.txt

      - name: List files
        run: ls
```

After the workflow runs, the file **runner-test.txt** appears on my machine, proving the job executed on my local runner.

---

# Task 5 – Runner Labels

Labels help target specific runners when multiple self-hosted runners are available.

Example label:

```
my-linux-runner
```

Workflow example:

```yaml
runs-on: [self-hosted, my-linux-runner]
```

This ensures the job runs only on runners with that label.

### Why labels are useful

Labels allow better control over which machine executes the job.

For example:

* GPU runner
* High-memory runner
* Docker-enabled runner
* Linux-specific runner

This is especially useful in large CI/CD environments.

---

# Task 6 – GitHub-Hosted vs Self-Hosted Runners

| Feature             | GitHub-Hosted Runner               | Self-Hosted Runner                           |
| ------------------- | ---------------------------------- | -------------------------------------------- |
| Who manages it      | GitHub                             | Developer / Organization                     |
| Cost                | Free minutes or paid usage         | Cost of your own server or VM                |
| Pre-installed tools | Many tools already installed       | Must install manually                        |
| Good for            | Standard CI/CD pipelines           | Custom infrastructure and advanced workloads |
| Security concern    | Code runs on GitHub infrastructure | You must manage security and updates         |

---

# Key Learnings

Today I learned:

* What **GitHub Actions runners** are
* How **GitHub-hosted runners** work
* How to configure a **self-hosted runner**
* How to run workflows on my own machine
* How **runner labels** help manage multiple runners
* Differences between **GitHub-hosted and self-hosted runners**

