# DevOps Engineering Report: Advanced Git & CI/CD Strategy

**Prepared by:** Simon  
**Topic:** Workflow Protection, Commit Rebase, Cherry-Picking, and Automated CI Testing

---

## 1. Workflow Architecture & Protection

### Selected Strategy: Trunk-Based Development
For our fast-moving development environment, **Trunk-Based Development** is our core strategy. It simplifies team velocity by avoiding complex branching structures.
* **Mechanism:** Developers write code on short-lived feature branches and merge them into the central `main` trunk frequently via monitored Pull Requests.
* **Key Benefits:**
  * Drastically reduces merge conflicts and integration issues.
  * Accelerates deployment times and deployment speed.
  * Enhances day-to-day team communication and engineering collaboration.
  * Provides native compatibility with Continuous Integration and Continuous Deployment (CI/CD) practices.

### Branch Protection Guidelines for `main`
To ensure production stability and stop broken builds from hitting the live servers, we enforce these GitHub Branch Protection Rules:
1. **Enforce PR Reviews:** Require at least one formal peer review approval before merging code.
2. **Mandate Status Checks:** Require all automated CI pipeline tests to pass successfully before the pull request can be merged.
3. **Block Unchecked Access:** Strictly prevent direct git pushes to the protected `main` branch.
4. **Maintain Freshness:** Require feature branches to be completely up to date with `main` before execution.

---

## 2. History Cleanup (Interactive Rebase)

### Command Operations: `pick` vs `squash`
Maintaining a clean git history is crucial before opening a Pull Request. During an interactive rebase, we leverage these two key commands:
* **`pick`:** Retains the target commit precisely as it is without alterations.
* **`squash`:** Condenses and merges the changes of the selected commit into the immediately preceding commit.

Using `squash` allows our engineering team to consolidate scattered development commits (e.g., combining `wip1`, `wip2`, `wip3`, `typo`, and `done`) into a single, clean production commit.

### How to Initiate Interactive Rebase
To clean up and manage the last 5 commits, run the following command in your terminal:

```bash
git rebase -i HEAD~5


3. The Emergency Surgery (Cherry-Picking)
Strategic Utility of git cherry-pick
The git cherry-pick command isolates and copies a single specific commit via its unique hash from one branch, applying it directly onto another branch without initiating a bulk merge.

This provides the perfect hotfix tool during production emergencies. It allows us to deploy a critical patch from develop directly to main without risking contamination from surrounding, untested code.


# Move to the production branch
git checkout main

# Synchronize with the latest upstream production changes
git pull origin main

# Apply only the verified hotfix commit
git cherry-pick 8f3a9b2

# Push the isolated fix back up to production
git push origin main



4. Automated Integration Testing & Docker CI (GitHub Actions)
The Value of Automated Pipelines
Running integration tests directly inside a centralized CI pipeline ensures every single code iteration runs within an entirely pristine, isolated container environment. Utilizing Docker Compose validates that multi-container architectures (e.g., Database, API layer, and Web frontends) maintain flawless internal networking before code reaches main. This architecture permanently eliminates the traditional "works on my machine" development blocker.



name: Build and Deploy Application Container

run-name: Deploying Container Release via GitHub Actions

on:
  workflow_dispatch:
    inputs:
      image-version:
        description: 'Specify the application build version tag'
        type: string
        required: true

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Securely pull repository source code
      - name: Fetch Source Code
        uses: actions/checkout@v4

      # Step 2: Establish connection to remote container registry
      - name: Authenticate with Docker Hub
        uses: docker/login-action@v3
        with:
          username: simondev
          password: ${{ secrets.SIMON_DOCKER_TOKEN }}

      # Step 3: Build container from local context and upload image
      - name: Compile and Push Container Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: simondev/web-app:${{ inputs.image-version }}