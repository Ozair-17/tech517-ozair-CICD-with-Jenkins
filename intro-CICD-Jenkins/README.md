# Intro to Jenkins & CI/CD

This document gives an overview of Jenkins and how it fits into modern CI/CD workflows. It also touches on the business value of pipelines, the role of webhooks, and the importance of automation in modern software development.

---

## What is CI? Benefits?
**CI (Continuous Integration)** is the practice of automatically integrating code changes from multiple developers into a shared repository several times a day. Each integration is verified by an automated build and test process.

### Benefits:
- **Catch bugs early** through frequent testing.
- **Faster feedback** to developers.
- **Reduces integration issues** and conflicts.
- **Improves code quality** and consistency.
- **Shortens release cycles**.

---

## What is CD? Benefits?
**CD** refers to two related practices:
- **Continuous Delivery**: Automatically prepares code for release to production.
- **Continuous Deployment**: Automatically deploys every change that passes tests to production.

### Benefits:
- **Faster time-to-market**.
- **Reliable, repeatable deployments**.
- **Reduced manual effort**.
- **Improved productivity and developer morale**.
- **Rapid response to user feedback**.

---

## What is Jenkins?
**Jenkins** is an open-source automation server used to implement CI/CD pipelines. It supports building, testing, and deploying code through customizable jobs and plugins.

- Written in Java
- Web-based GUI + CLI
- Supports pipelines via Groovy syntax (Jenkinsfile)
- Large plugin ecosystem
- Works with Git, Docker, Kubernetes, Terraform, etc.

---

## Why use Jenkins? Benefits of using Jenkins? Disadvantages?

### Benefits:
- Free and open-source
- Highly extensible via plugins
- Integrates with most development and cloud tools
- Active community
- Easy to set up and use for small and large teams

### Disadvantages:
- Plugin compatibility issues
- Can become difficult to maintain at scale
- UI can be clunky and outdated
- Requires proper security configuration

---

## Stages of Jenkins
A Jenkins pipeline typically includes the following stages:

1. **Checkout** – Pull code from Git or other SCM.
2. **Build** – Compile or package the code.
3. **Test** – Run unit, integration, or system tests.
4. **Static Analysis** – Code linting, vulnerability scans.
5. **Deploy** – Deploy to staging or production.
6. **Notify** – Send status updates via email, Slack, etc.

These are defined in a `Jenkinsfile` and can be triggered automatically.

---

## What alternatives are there for Jenkins?
- **GitHub Actions** – Integrated with GitHub, easy for small/medium projects.
- **GitLab CI/CD** – Fully integrated with GitLab.
- **CircleCI** – Cloud-based, focuses on simplicity and speed.
- **Travis CI** – Popular in open source, integrates with GitHub.
- **Azure DevOps Pipelines** – For teams using Microsoft stack.
- **TeamCity**, **Drone CI**, **Bamboo**, etc.

---

## Why build a pipeline? Business value?

- **Faster Delivery** – Deploy features and fixes quickly.
- **Reduced Risk** – Automated testing and rollbacks.
- **Scalability** – Repeatable process across teams and projects.
- **Cost Efficiency** – Less time spent on manual tasks.
- **Auditability** – Logs and status tracking for compliance.
- **Improved Developer Workflow** – More time to focus on features.

---

## What is Deployment vs Delivery?

| Term | Meaning |
|------|---------|
| **Continuous Delivery** | Code is always in a deployable state. Manual approval to deploy. |
| **Continuous Deployment** | Code is automatically deployed after passing tests. No manual steps. |
| **Deployment** | Actual release of code to staging or production environments. |

---

## What is a Webhook?
A **webhook** is a way for one system to notify another system when an event occurs.

### In CI/CD:
- A **Git webhook** triggers Jenkins to start a build whenever code is pushed to the repo.

```json
{
  "ref": "refs/heads/main",
  "repository": {
    "name": "my-app"
  },
  "pusher": {
    "name": "dev123"
  }
}
```

### Example Flow:
1. Dev pushes code to GitHub
2. Webhook triggers Jenkins
3. Jenkins builds/tests/deploys automatically

---

## Bonus: Jenkinsfile Example
```groovy
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/example/my-repo.git'
      }
    }
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
    stage('Deploy') {
      steps {
        sh './deploy.sh'
      }
    }
  }
}
```
