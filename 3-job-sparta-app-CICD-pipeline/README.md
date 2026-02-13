- [Jenkins CI/CD Configuration – Sparta App](#jenkins-cicd-configuration--sparta-app)
  - [Overview](#overview)
- [CI - Running tests on new push](#ci---running-tests-on-new-push)
    - [Build Retention (Discard Old Builds)](#build-retention-discard-old-builds)
    - [GitHub Project Link](#github-project-link)
  - [Source Code Management (Git Setup)](#source-code-management-git-setup)
    - [Repository Configuration](#repository-configuration)
    - [Credentials](#credentials)
    - [Branch Specification](#branch-specification)
  - [Build Triggers (Automation)](#build-triggers-automation)
    - [GitHub Webhook Trigger](#github-webhook-trigger)
  - [Build Environment Configuration](#build-environment-configuration)
    - [Node.js Installation](#nodejs-installation)
    - [npm Configuration](#npm-configuration)
    - [npm Cache](#npm-cache)
  - [Build Steps Explained](#build-steps-explained)
    - [Step 1 – Navigate to Application Directory](#step-1--navigate-to-application-directory)
    - [Step 2 – Install Dependencies](#step-2--install-dependencies)
    - [Step 3 – Run Tests](#step-3--run-tests)
- [Merge](#merge)
  - [Build Trigger Configuration](#build-trigger-configuration)
  - [Post-Build Git Publishing](#post-build-git-publishing)
    - [Push Only If Build Succeeds](#push-only-if-build-succeeds)
    - [Merge Results Enabled](#merge-results-enabled)
    - [Branch Configuration](#branch-configuration)
- [Deploy](#deploy)
  - [Build Environment – SSH Agent Setup](#build-environment--ssh-agent-setup)
  - [Deployment Script Breakdown](#deployment-script-breakdown)
    - [1. Defining the Target Server](#1-defining-the-target-server)
    - [2. Copying Application Code to EC2](#2-copying-application-code-to-ec2)
    - [3. Remote Application Setup via SSH](#3-remote-application-setup-via-ssh)
    - [4. Installing Dependencies](#4-installing-dependencies)
    - [5. Application Process Management with PM2](#5-application-process-management-with-pm2)
  - [6. Possible Blockers/Improvements to keep an eye out for](#6-possible-blockersimprovements-to-keep-an-eye-out-for)


# Jenkins CI/CD Configuration – Sparta App

## Overview

This document explains the Jenkins CI configuration I created for my Sparta App project. The goal of this setup is to automate the process of pulling the latest code from GitHub, installing dependencies, running tests, and ensuring that any new changes are validated automatically.

This is essentially a Continuous Integration (CI) pipeline. It ensures that every code update is tested and verified without manual intervention. While this configuration currently focuses on CI, it lays the foundation for full Continuous Deployment (CD) in the future.

---

# CI - Running tests on new push

### Build Retention (Discard Old Builds)
![alt text](<../images/Screenshot 2026-02-12 at 16.27.18.png>)
I enabled log rotation and limited Jenkins to keeping only the last five builds.

This is important because:

* Jenkins can consume a lot of disk space over time.
* Old logs usually aren’t needed after debugging is complete.
* Keeping recent builds makes troubleshooting easier without clutter.

This is a standard best practice in CI/CD environments.

### GitHub Project Link
![alt text](<../images/Screenshot 2026-02-12 at 16.27.29.png>)
I added the GitHub repository URL so Jenkins references the source project directly. This does not affect functionality but improves visibility and navigation for team members.

---

## Source Code Management (Git Setup)

### Repository Configuration

Jenkins is configured to pull code from my GitHub repository using Git over SSH:

* This is more secure than HTTPS authentication.
* It avoids username/password prompts.
* It is standard practice for automated CI systems.

### Credentials

I created and configured an SSH key within Jenkins credentials.

This allows Jenkins to:

* Clone the repository.
* Pull updates automatically.
* Interact securely with GitHub.

Without this step, Jenkins would not be able to access the repository reliably.

### Branch Specification

The job is set to build only the `main` branch.

This ensures:

* Only stable code triggers builds.
* Feature branches don’t accidentally deploy or run CI unless intended.

In a larger team environment, additional branch rules could be added later.

---

## Build Triggers (Automation)
![alt text](<../images/Screenshot 2026-02-12 at 16.27.39.png>)
### GitHub Webhook Trigger

I enabled the GitHub hook trigger for Git SCM polling.

This means:

* Whenever I push code to GitHub, GitHub sends a webhook notification to Jenkins.
* Jenkins automatically starts a build.

This eliminates the need for manual builds or scheduled polling.

It ensures:

* Faster feedback on code changes.
* Immediate testing.
* True Continuous Integration behaviour.

---

## Build Environment Configuration

### Node.js Installation

Since the application is a Node.js app, I configured Jenkins to use Node.js version 20.

This step ensures:

* Node and npm are available during builds.
* Consistent runtime environment.
* Compatibility with the project dependencies.

This avoids version mismatch issues that often cause CI failures.

### npm Configuration

I left the npm configuration as system default because:

* No private registry is currently used.
* No authentication tokens are required.

If private packages were introduced later, this section would be updated.

### npm Cache

Default caching is enabled. This speeds up builds because previously downloaded packages don’t need to be fetched again unless necessary.

---

## Build Steps Explained
![alt text](<../images/Screenshot 2026-02-12 at 16.27.46.png>)
The core CI logic happens in the shell build step.

### Step 1 – Navigate to Application Directory

```
cd Sparta-App
```

The repository contains multiple folders, so Jenkins needs to move into the actual application directory before running commands.

### Step 2 – Install Dependencies

```
npm install
```

This installs all required Node.js dependencies listed in `package.json`.

This includes things like:

* Express or backend libraries
* Database connectors
* Testing frameworks

This ensures the app environment matches development conditions.

### Step 3 – Run Tests

```
npm test
```

This runs the automated test suite.

This is crucial because:

* It validates new code changes.
* Prevents broken code from progressing further.
* Provides quick developer feedback.

If tests fail, Jenkins marks the build as failed.

---

# Merge



## Build Trigger Configuration
![alt text](<../images/Screenshot 2026-02-12 at 16.39.32.png>)
This merge job is configured to start only after the test job completes successfully.

Specifically:

* The job watches the previous Jenkins job (`Ozair-job1-test`).
* It triggers only if that job is stable (tests passed).

This ensures that code merging only happens after validation.

This is an important CI/CD safety mechanism because merging failed or unstable code could break production pipelines.

---

## Post-Build Git Publishing
![alt text](<../images/Screenshot 2026-02-12 at 16.36.36.png>)
This job uses Jenkins Git Publisher functionality to push results back to the repository.

### Push Only If Build Succeeds

Enabled to ensure:

* Failed builds never modify the repository.
* Only verified code gets merged.

This protects the integrity of the main branch.

### Merge Results Enabled

This allows Jenkins to:

* Merge the tested branch changes.
* Push those merged results back to the origin repository.

Effectively, Jenkins is acting as an automated gatekeeper for merging.

This removes manual merge steps and reduces human error.

### Branch Configuration

Configured values:

* Branch to push: `main`
* Target remote: `origin`

This means Jenkins pushes validated code directly to the main branch on the GitHub repository.

---

# Deploy

## Build Environment – SSH Agent Setup
![alt text](<../images/Screenshot 2026-02-13 at 09.27.47.png>)
This job uses the Jenkins SSH Agent plugin with an EC2 private key credential.

This allows Jenkins to:

* Authenticate securely to the EC2 instance
* Execute remote commands via SSH
* Transfer application files via SCP

Using managed credentials is important because:

* Secrets are stored securely inside Jenkins
* Keys are not exposed in scripts or repositories
* Access can be rotated easily

This mirrors standard infrastructure automation practices.

---

## Deployment Script Breakdown
![alt text](<../images/Screenshot 2026-02-13 at 09.27.56.png>)
The deployment is handled through a shell execution step in Jenkins.

### 1. Defining the Target Server

The script defines an EC2 public IP variable. This identifies the target server where the application will be deployed.

This makes the script easier to update if infrastructure changes.

---

### 2. Copying Application Code to EC2

The script uses SCP to copy the Sparta App directory to the EC2 instance.

Key points:

* Strict host key checking is disabled to prevent interactive prompts during automation.
* Files are transferred directly to the remote user’s home directory.
* This ensures the latest build artifacts are deployed.

This replaces manual file transfer steps typically done during development.

---

### 3. Remote Application Setup via SSH

After copying the code, Jenkins connects to the EC2 server via SSH and runs commands remotely.

These commands include:

* Navigating to the application directory
* Installing Node.js dependencies
* Managing the application process

This ensures the server environment matches the latest code version.

---

### 4. Installing Dependencies

The deployment runs npm install on the server.

This step:

* Installs required packages
* Ensures compatibility with the latest code
* Avoids missing dependency errors during runtime

It mirrors the CI testing environment but executes on the live server.

---

### 5. Application Process Management with PM2

PM2 is used to manage the Node.js application process.

The script performs:

* Safe deletion of any existing PM2 process
* Restart of the application using npm start

Benefits of PM2 include:

* Automatic process restart if the app crashes
* Background process management
* Logging and monitoring capabilities

This ensures application stability after deployment.

## 6. Possible Blockers/Improvements to keep an eye out for

* Webhook into the the right git
* Using the correct IP addreses
* Correct Keys 
* Making sure all dependencies are installed for the app
* If reverse proxy not set up then need 
* Correct path when executing/deploying the app
* Using rsync over scp - 'Syncronises' changes (Only copies over changes from last time)

