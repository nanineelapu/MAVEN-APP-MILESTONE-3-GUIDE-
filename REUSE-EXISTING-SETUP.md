# Reuse Your Existing 3 Instances — DO NOT Create New Ones

You already built and ran the **e-commerce** Milestone 3 pipeline. That means your
infrastructure is **already done**. For this Maven case study you reuse the exact same
servers and tools. This page tells you the ONLY things that change.

---

## What you ALREADY have (keep as-is)

| Instance | Tool | Already installed? | Action |
|---|---|---|---|
| 1 | Jenkins + Java + Maven + Git + Ansible | ✅ Yes | **Reuse** |
| 2 | SonarQube | ✅ Yes | **Reuse** (just add a new project) |
| 3 | Apache Tomcat | ✅ Yes | **Reuse** (new WAR gets deployed here) |

✅ **No new EC2 / VM instances.**
✅ **No reinstalling Jenkins, SonarQube, Tomcat, Maven, or Ansible.**
✅ **No new SSH keys** — the Jenkins → Tomcat SSH trust you set up still works.

---

## What is NEW / changes for the Maven app

Only **5 small things** change vs. the e-commerce run:

### 1. New GitHub repo for the Maven application
The instructor gives you a **Maven web app** repo (produces a `.war`).
When you receive it, just set its URL in the Jenkinsfile (`APP_REPO`) and in the
Jenkins job. Until then use the placeholder.

### 2. New SonarQube project key
So the Maven analysis doesn't overwrite the e-commerce report.
- Reuse the **same SonarQube token** (Jenkins credential `sonar-token`) — no need to regenerate.
- Just use a new `-Dsonar.projectKey=maven-web-app` in the build.
  (SonarQube auto-creates the project on first analysis.)

### 3. New Jenkins pipeline job
- Jenkins → **New Item** → name it `maven-web-app-pipeline` → **Pipeline**.
- "Pipeline script from SCM" → Git → this guide repo (or paste the Jenkinsfile).
- Everything else (tool config, credentials, SonarQube server) is already configured globally — reuse it.

### 4. The WAR file name / Tomcat context changes
- E-commerce deployed e.g. `ecommerce.war`. Maven app deploys e.g. `maven-web-app.war`.
- Update the `WAR_NAME` variable in [Jenkinsfile](Jenkinsfile) and [deploy.yml](deploy.yml).
- App URL becomes `http://<TOMCAT_IP>:8080/<WAR_NAME-without-.war>/`.

### 5. Maven build command (this is a real Maven project, so it's clean)
- Build: `mvn clean package` → output in `target/*.war`.

---

## 5-minute pre-flight checklist (before you build)

Run these to confirm the reused servers are healthy:

```bash
# On Jenkins (Instance 1)
sudo systemctl status jenkins      # active (running)
mvn -version                       # Maven present
ansible --version                  # Ansible present
java -version                      # Java present

# SonarQube reachable (Instance 2) — from Jenkins box
curl -I http://<SONAR_IP>:9000     # HTTP 200

# Tomcat reachable (Instance 3)
curl -I http://<TOMCAT_IP>:8080    # HTTP 200

# Ansible can reach Tomcat over SSH (from Jenkins box)
ansible -i hosts.txt tomcat -m ping   # SUCCESS / pong
```

If all four pass, you do not touch the infrastructure — just run the new pipeline.

---

## Existing Jenkins config you reuse (Manage Jenkins → Tools / Credentials)

| Item | Where | Reuse? |
|---|---|---|
| JDK installation | Tools | ✅ |
| Maven installation (e.g. `maven3`) | Tools | ✅ |
| SonarQube server config + `sonar-token` | System + Credentials | ✅ |
| `SonarQube Scanner` plugin | Plugins | ✅ |
| GitHub credential (if private repo) | Credentials | ✅ (or add Maven repo) |
| SSH key for Tomcat (Ansible) | On Jenkins box `~/.ssh` | ✅ |

> Bottom line: **copy your e-commerce setup, swap the repo URL + project key + WAR name, run.**
