# Theory Deliverables — Maven Web App CI/CD (All 6 Required Submissions)

This document answers every deliverable the case study asks for. Paste into your
report / PPT and attach the screenshots from [SCREENSHOTS-GUIDE.md](SCREENSHOTS-GUIDE.md).

---

## Deliverable 1 — CI/CD Architecture Diagram
See [ARCHITECTURE.md](ARCHITECTURE.md). Components:
- **Developer** → commits to GitHub.
- **GitHub Repository** → stores Maven source + `pom.xml`.
- **CI/CD Server (Jenkins, Instance 1)** → orchestrates the 5 stages; also hosts the
  **Maven** build tool and the **Ansible** controller.
- **Code Quality Server (SonarQube, Instance 2)** → static analysis + Quality Gate.
- **Build Process (Maven)** → compiles, runs lifecycle, produces the **WAR**.
- **Configuration Management Server (Ansible)** → automates Tomcat lifecycle + deploy.
- **Application Server (Apache Tomcat, Instance 3)** → hosts the running web app.

---

## Deliverable 2 — Complete Pipeline Design

| Stage | Objective | Implementation | Pass/Fail behaviour |
|---|---|---|---|
| 1. Checkout | Fetch latest code | `git` step, commit logged | fails fast if repo/branch unreachable |
| 2. Static Analysis | Code quality | `mvn clean compile sonar:sonar` | analysis pushed to SonarQube |
| 3. Quality Gate | Enforce quality | `waitForQualityGate abortPipeline:true` | FAIL → pipeline aborts, no build |
| 4. Build | Produce artifact | `mvn clean package` → `target/*.war` | fails if compile/package fails |
| 5. Deploy | Ship to Tomcat | `ansible-playbook deploy.yml` | stop→copy→start→verify |

Declarative Jenkinsfile (see [Jenkinsfile](Jenkinsfile)). Build history is retained
via `buildDiscarder` (last 10) and artifacts via `archiveArtifacts`.

---

## Deliverable 3 — Tool Integration Flow
- **Jenkins ↔ GitHub:** SCM checkout (credentialed). Optional webhook for auto-trigger.
- **Jenkins ↔ SonarQube:** Maven Sonar plugin + `withSonarQubeEnv`; gate result returned
  to Jenkins via SonarQube webhook consumed by `waitForQualityGate`.
- **Jenkins ↔ Ansible:** Jenkins invokes `ansible-playbook` on the same box (controller).
- **Ansible ↔ Tomcat:** SSH (key-based) — copies WAR, controls the `tomcat` service.
See the integration table in [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Deliverable 4 — Deployment Strategy
- **Recreate / replace strategy:** stop Tomcat → remove old WAR + exploded dir → copy
  new WAR → start Tomcat. Simple, deterministic, matches "replace existing deployment".
- **Only successful builds deploy** — Quality Gate + build success are prerequisites.
- **Idempotent:** re-running the playbook always converges to the latest WAR.
- **Validation:** post-deploy HTTP 200 health check before declaring success.
- *Upgrade path:* blue-green / parallel Tomcat contexts or a load balancer for zero-downtime
  (see High Availability below).

---

## Deliverable 5 — Infrastructure Automation Approach
- **Ansible** is the Configuration Management tool (declarative `deploy.yml`).
- Manages **server state**: service stopped/started, files present/absent.
- **No manual steps:** no SSH login, no manual Tomcat restart, no manual WAR copy,
  no manual config edits — all in the playbook.
- **Inventory** (`hosts.txt`) controls *which* servers and *how* to connect.
- Re-usable across environments by swapping the inventory (dev/stage/prod).

---

## Deliverable 6 — Failure Handling Strategy

| Concern | Approach |
|---|---|
| Quality Gate fails | `abortPipeline: true` stops everything; Tomcat keeps the old, known-good WAR |
| Build fails | Pipeline stops at Stage 4; nothing is deployed |
| Deploy fails | `post { failure }` reports it; previous WAR remains because copy/start is the last step; health check `until 200` catches a bad start |
| Rollback | Keep the previous WAR artifact (archived in Jenkins / fingerprinted). Re-run the deploy playbook with `war_src=<previous WAR>`. Optionally keep `webapps/app.war.bak` |
| Minimise downtime | Tomcat restart window is seconds; for zero-downtime use 2 Tomcat nodes behind a load balancer and deploy one at a time (blue-green) |
| Logs | Jenkins **Console Output** + **Stage View** per build; Ansible verbose (`-v`); Tomcat `catalina.out`; SonarQube analysis history |

---

## Security Requirements (addressed)
- **Secure credential management:** SonarQube token and SSH key stored in Jenkins
  Credentials / on the controller — **never** in the Jenkinsfile.
- **No hardcoded passwords:** all secrets referenced by credential ID / key path.
- **Secure server-to-server comms:** SSH (key-based) for Ansible→Tomcat; HTTPS where available.
- **Controlled deployment access:** only the Jenkins job (service account) can trigger deploys.
- **Least privilege:** SSH user limited to Tomcat ops; Sonar token is analysis-scoped.

## High Availability Considerations
- **Deployment failures:** detected by the post-deploy health check; build marked FAILURE;
  old version still serving.
- **Rollback plan:** redeploy the last good archived WAR via the same playbook (one command).
- **Minimised downtime:** fast restart; blue-green/LB option documented for zero-downtime.
- **Log retention:** Jenkins build history (10 builds), archived artifacts, Tomcat logs,
  Sonar analysis history — full audit trail.
