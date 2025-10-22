# Infinion_DevOps_Assessment

## Self-Hosted GitHub Actions Runner Setup

## Overview
This repository documents the process of setting up and configuring a self-hosted GitHub Actions runner locally on a Linux machine. The goal of this assessment is to demonstrate hands-on understanding of CI/CD using GitHub Actions with a self-hosted runner.

## Setup Instructions

### Prerequisites
- [Git](https://git-scm.com/) # installed locally
- GitHub account with repository access
- Linux Machine

### Installation Steps

**Step1:** Installed git locally

`sudo apt-get install git`

**Step2:** Created a github repository, cloned the repo locally and created a `README.md` file to document all necessary steps involved in doing this task.

**Step3** Navigated to Settings → Actions → Runners → New self-hosted runner, i am currently using a linux machine, so i selected a Linux as my runner image. Followed these steps to create a runner image locally;

```bash
# Create a folder
$ mkdir actions-runner && cd actions-runnerCopied!# Download the latest runner package

# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.329.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.329.0/actions-runner-linux-x64-2.329.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz

# Create the runner and start the configuration experience
./config.sh --url https://github.com/PreciousDipe/Infinion_DevOps_Assessment --token your-token

# Last step, run it!
./run.sh
```
![](/assets/1.png)

![](/assets/2.png)

**Step4** Next thing i did was to create a workflow here `.github/workflow/test.yml` to test the runner

```yaml
name: Test Local Self-Hosted Runner

on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

jobs:
    test:
        runs-on: self-hosted
        steps:
            - uses: actions/checkout@v4

            - name: Display System Info
              run: |
                  echo "OS: $(uname -s || ver)" # 
                  echo "Current User: $(whoami)" # gets the name of the current user
                  echo "Working Directory: $(pwd)" # gets the working directory

            - name: Run a simple test
              run: |
                  echo "Hello from self-hosted runner!"
                  echo "Testing basic commands..."
                  node --version || echo "Node not installed" # checks if node is installed

            - name: Create test artifact
              run: |
                  echo "Test passed!" > test-result.txt # creates a file saying test passed

            - name: Upload artifacts
              uses: actions/upload-artifact@v4
              with:
                  name: test-results
                  path: test-result.txt
```

**Step5** I then created a .gitignore file to add things i dnt want to push, like the files in the `actions-runner` directory. Updated my `Readme.md` file then pushed my code to my repository.

**Step6** Tested the workflow and ensured all tests pass, then checked that the job process running locally shows all processes ended successfully.
![](/assets/3.png)

## Challenges Faced and Solutions

Using a deprecated version of an action
**Problem**: My workflow failed because i used a deprecated version of this action `actions/upload-artifact`
**Solution**:
- Went to the github Market place to search for the latest version of this action `actions/upload-artifact` then updated the version from `v3` to `v4`

## Things i would do differently in a production environment?

1. I'll use an infrastructure as code (IAC) tool to automatically set up, configures, and maintains all runners. To make the process reproducible and error prone.

2. Set up Monitoring systems like Prometheus and Grafana or DataDog, that continuously monitors the runner health, collect metrics, and sends alerts when something goes wrong with the runner.

3. I'll regularly backup runner configurations and workflow artifacts to safe storage like S3 and Google Cloud Storage depending on your cloud provider.

4. I'll setup up the runner on a virtual machine and depending on the project consider using multiple runners across different virtual machines. Each runner can pick up jobs independently, and GitHub's queue system automatically distributes work evenly.

## Security considerations I implemented

1. I used  official, trusted GitHub Actions from the marketplace like `actions/checkout@v4` and `actions/upload-artifact@v4`. I pinned them to specific versions instead of using @latest to prevent supply chain attacks where a maintainer could update the action maliciously.

2. I ensured artifacts only contain non-sensitive data by deliberately excluding system information and environment variables from the artifact file. In my workflow, I created `test-result.txt` with only "Test passed!" output, no tokens, API keys, or credentials. All artifact uploads happen over HTTPS encryption, and even if the artifact is downloaded, it contains no exploitable information.

3. I used a self-hosted runner that communicates with GitHub strictly over HTTPS, meaning all workflow data, job instructions, and tokens are encrypted in transit. The runner executes all workflow steps as a non-root user, which limits damage if the workflow is compromised. SSH root login is disabled on my PC, preventing attackers from gaining administrative access to read the runner's configuration file containing the GitHub token.

4. I restricted the workflow to only trigger on the main branch (push: branches: [main] and pull_request: branches: [main]), preventing unauthorized feature branches or forked repositories from executing builds. This reduces the attack surface by ensuring only production-ready code branches trigger the CI/CD pipeline.