# 13. Jenkins CI/CD — Real Build Failures & Fixes

Two real pipeline failures I hit while setting up Jenkins CI/CD for the two-tier Flask + MySQL app, and what actually caused each one.

---

## 🐛 Issue 1: Docker Build Failed — "no permission to read" from `mysql-data`

### The Error
```
+ docker build -t two-tier-flask-app .
checking context: no permission to read from
'/var/lib/jenkins/workspace/cicd-pra/mysql-data/auto.cnf'
```
Build stage failed immediately, which cascaded into `test`, `Push`, and `Deploy` all being skipped.

### Root Cause
My repo contained a `mysql-data/` folder — the **bind-mounted MySQL data directory** from running `docker compose up` locally (MySQL writes its database files like `auto.cnf` into this folder at runtime).

When Jenkins ran `docker build .`, Docker tries to read **everything** in the build context (the current directory) to send it to the Docker daemon — including `mysql-data/`. But those files are owned by the **MySQL container's internal user** (a UID that doesn't correspond to a real user on the Jenkins host), so the `jenkins` user running the build had no permission to read them.

### The Fix
Database data should never be part of a Docker **build context** in the first place — it's runtime data, not application code. Two-part fix:

**1. Exclude it via `.dockerignore`:**
```
mysql-data/
*.pyc
__pycache__/
.git
```

**2. Make sure `mysql-data/` isn't committed to the repo at all** (it should already be running as a Docker volume/bind mount outside version control):
```
# .gitignore
mysql-data/
```

### What I Learned
- `docker build .` sends the **entire current directory** to the Docker daemon as build context unless told otherwise — anything in that folder is fair game, whether you meant to include it or not.
- Local runtime artifacts (database files, logs, `node_modules`, `.env` files) should almost always be excluded via `.dockerignore` — not just for permissions, but because they bloat the build context and can leak sensitive data into an image.
- This error only ever showed up on Jenkins (not locally) because my local user owned those files, but Jenkins runs builds as its own `jenkins` system user — a good reminder that CI environments don't inherit your local machine's permissions.

---

## 🐛 Issue 2: Docker Push Failed — "repository name must be lowercase"

### The Error
```
+ docker login -u dextern001 -p ****
Login Succeeded

+ docker image tag two-tier-flask-app dextern001/two-tier-flask-app

+ docker push ****/two-tier-flask-app:latest
repository name must be lowercase
```

Login succeeded, tagging succeeded (using the literal username `dextern001`) — but the **push** step failed, and Jenkins masked part of the image name with `****` in the log, which was the actual clue.

### Root Cause
Jenkins automatically masks any console output that matches a registered credential's value (that's the `****`). The fact that the **push** command triggered masking — but the earlier `docker image tag` command didn't — meant the push step was referencing a **credential variable**, not the plain-text username I'd used for tagging.

In my pipeline script, the push stage used the **password** credential variable instead of the **username** variable to build the image path:
```groovy
// ❌ Wrong — uses the password credential as the Docker Hub namespace
sh "docker push ${dockerHubPass}/two-tier-flask-app:latest"
```
Since Docker Hub requires repository names to be **all lowercase**, and my Docker Hub password contains uppercase characters and symbols, Docker correctly rejected it as an invalid repository name — the error was accurate, just confusing until I realized *what* was actually being substituted into that string.

### The Fix
Use the correct credential variable, and reference the **same tag** used during the `docker image tag` step:
```groovy
withCredentials([usernamePassword(
    credentialsId: 'dockerhub-creds',
    usernameVariable: 'dockerHubUser',
    passwordVariable: 'dockerHubPass'
)]) {
    sh """
        echo \$dockerHubPass | docker login -u \$dockerHubUser --password-stdin
        docker image tag two-tier-flask-app \$dockerHubUser/two-tier-flask-app:latest
        docker push \$dockerHubUser/two-tier-flask-app:latest
    """
}
```
Also switched from `-p ****` (passing the password directly as a CLI flag) to `--password-stdin`, since Jenkins itself warned:
```
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! A secret was passed to "sh" using Groovy String interpolation, which is insecure.
```

### What I Learned
- When Jenkins masks part of a log line with `****`, that's not just noise — it's telling you a **credential variable was used right there**, which is often the fastest way to spot a variable mix-up.
- Docker Hub repository names (the `namespace/image` part) must be lowercase — if you ever see this error, check *what value is actually being substituted* into that string rather than assuming your username is the problem.
- `--password-stdin` isn't just a style preference — passing passwords via `-p` on the CLI can leak them into shell history and process lists on the Jenkins agent; `--password-stdin` avoids that.
- Always reference credentials by their **intended purpose** (`dockerHubUser` for the namespace, `dockerHubPass` only for authentication) — a copy-paste of the wrong variable name is an easy mistake that produces a very unintuitive error message.

---

## 🧰 Commands / Checks Used While Debugging
```bash
docker build -t two-tier-flask-app .          # reproduced build failure locally
ls -la mysql-data/                            # found ownership mismatch
cat .dockerignore                             # confirmed mysql-data/ wasn't excluded
docker login -u dextern001 -p ****            # confirmed login itself worked
docker image tag two-tier-flask-app dextern001/two-tier-flask-app
docker push dextern001/two-tier-flask-app:latest   # worked once corrected manually
```

## 📌 Key Takeaway
Both failures had accurate, specific error messages — the hard part wasn't Docker or Jenkins being unclear, it was **reading precisely what each error said** instead of assuming the obvious cause. "No permission to read" meant exactly that (a file ownership issue, not a Docker bug), and "repository name must be lowercase" meant exactly that (check what's actually in that string) — not a login or Docker Hub account problem.
