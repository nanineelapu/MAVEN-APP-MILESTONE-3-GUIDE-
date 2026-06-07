# Enterprise CI/CD Pipeline — Java Maven Web Application (Milestone 3)

End-to-end **CI/CD pipeline** for a Java **Maven** web application that builds a **WAR**
artifact and deploys it to **Apache Tomcat** automatically — using the SAME 3 servers
you already built for the e-commerce case study. **You do NOT need to create new instances.**

> ⚠️ **Reuse, don't rebuild.** This case study uses the identical toolchain
> (Jenkins + SonarQube + Tomcat + Ansible). You only create a **new pipeline job**,
> a **new SonarQube project**, and point the pipeline at the **Maven app repo**.
> See [REUSE-EXISTING-SETUP.md](REUSE-EXISTING-SETUP.md) first.

---

## Toolchain (same as e-commerce)

| Case-study Server | Tool | Where it runs |
|---|---|---|
| CI/CD Server | **Jenkins** | Instance 1 |
| Code Quality Server | **SonarQube** | Instance 2 |
| Application Server | **Apache Tomcat** | Instance 3 |
| Configuration Management Server | **Ansible** | Controller on Instance 1 (Jenkins box) |

## Pipeline stages

```
Developer Commit
      ↓
1. Source Code Checkout      (Git / GitHub)
      ↓
2. Static Code Analysis      (SonarQube — bugs, vulns, smells)
      ↓
3. Quality Gate Decision     (PASS → continue | FAIL → stop)
      ↓
4. Application Build         (mvn clean package → target/*.war)
      ↓
5. Deployment Automation     (Ansible: stop Tomcat → copy WAR → start → verify)
      ↓
Service Validation           (curl health check)
```

## Files in this repo

| File | Purpose |
|---|---|
| [README.md](README.md) | This overview |
| [REUSE-EXISTING-SETUP.md](REUSE-EXISTING-SETUP.md) | **How to reuse your 3 existing instances** |
| [STEP-BY-STEP-COMMANDS.md](STEP-BY-STEP-COMMANDS.md) | Every command, copy-paste order |
| [Jenkinsfile](Jenkinsfile) | Declarative pipeline (the main method) |
| [ALTERNATIVE-FREESTYLE-METHOD.md](ALTERNATIVE-FREESTYLE-METHOD.md) | Freestyle-job alternative (no Jenkinsfile) |
| [deploy.yml](deploy.yml) | Ansible playbook — deploy WAR to Tomcat |
| [hosts.txt](hosts.txt) | Ansible inventory (Tomcat server) |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Architecture diagram + workflow |
| [THEORY-DELIVERABLES.md](THEORY-DELIVERABLES.md) | All 6 required deliverables (write-up) |
| [SCREENSHOTS-GUIDE.md](SCREENSHOTS-GUIDE.md) | Exactly which screenshots to capture |
| [completeFlow.txt](completeFlow.txt) | Quick end-to-end checklist |

## Quick start

1. Read [REUSE-EXISTING-SETUP.md](REUSE-EXISTING-SETUP.md) — confirm your 3 servers are up.
2. Replace `<MAVEN_APP_REPO_URL>` and IPs in [Jenkinsfile](Jenkinsfile), [hosts.txt](hosts.txt), [deploy.yml](deploy.yml).
3. Create a Jenkins **Pipeline** job → "Pipeline script from SCM" → this repo.
4. Click **Build Now**. Watch the 5 stages turn green.
5. Open `http://<TOMCAT_IP>:8080/<app-context>/` to verify.
