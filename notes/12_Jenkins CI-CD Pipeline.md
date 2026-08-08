## Why CI/CD After Docker?
Once an app is containerized, the natural next step is **automating** the build → test → push → deploy cycle so it happens on every code change instead of manually. Jenkins is one of the most widely used tools for this.

## Core Concepts

### Pipeline
A **Jenkins Pipeline** is a sequence of automated stages defined in code (a `Jenkinsfile`), instead of clicking through UI configuration. This makes the CI/CD process version-controlled alongside the application code.

### My Pipeline Stages
```
1. Code Checkout   → pull latest code from GitHub
2. Docker Build    → build the Docker image from the Dockerfile
3. Test            → run tests against the built image/app
4. Push to Docker Hub → tag and push the image to a registry
5. Deployment      → deploy using docker compose (or similar)
```

### Example `Jenkinsfile` (Declarative Pipeline)
```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = 'devashish/flask-mysql-app'
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/devashish-pandey/docker-learning.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run --rm $IMAGE_NAME:$BUILD_NUMBER pytest || true'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Deployment') {
            steps {
                sh 'docker compose down && docker compose up -d --build'
            }
        }
    }
}
```

---

## GitHub Webhook — Automatic Triggers
Instead of manually clicking "Build Now" in Jenkins, a **GitHub Webhook** notifies Jenkins automatically whenever code is pushed to the repository.

### Setup Steps
1. In Jenkins: install the **GitHub plugin**, and in the job config enable **"GitHub hook trigger for GITScm polling"**.
2. In GitHub repo: go to **Settings → Webhooks → Add webhook**.
3. Payload URL:
   ```
   http://<jenkins-server>:8080/github-webhook/
   ```
4. Content type: `application/json`
5. Event: **Just the push event**

Once configured, every `git push` to the repo triggers Jenkins to automatically pull the code and run the full pipeline — no manual trigger needed.

---

## Jenkins + Docker
Jenkins itself needs access to Docker to build/run images as part of the pipeline. Common setup:
- Run Jenkins in a container with the **Docker socket mounted**:
  ```bash
  docker run -d \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v jenkins_home:/var/jenkins_home \
    -p 8080:8080 \
    jenkins/jenkins:lts
  ```
  This lets the Jenkins container use the **host's Docker daemon** to build and run images, rather than running Docker-in-Docker.
- Install **Docker CLI** inside the Jenkins container/agent if it isn't already present.
- Jenkins can call `docker build`, `docker push`, and `docker compose up` directly from pipeline `sh` steps, just like running them manually from a terminal.

---

## Final Workflow
```
GitHub → Webhook → Jenkins → Build → Test → Docker Hub → Deploy
```

## Key Takeaway
A CI/CD pipeline isn't magic — it's the exact same manual commands (`docker build`, `docker push`, `docker compose up`) you'd run by hand, just automated and triggered by a webhook. Understanding the manual steps first (which I did with Docker basics) made it much easier to see what Jenkins is actually automating, rather than treating the pipeline as a black box.
