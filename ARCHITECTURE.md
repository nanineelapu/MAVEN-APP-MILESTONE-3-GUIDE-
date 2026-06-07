# CI/CD Architecture Diagram & Workflow — Maven Web Application

## Architecture Diagram (ASCII — copy into your report or redraw in draw.io)

```
                ┌────────────────┐
                │   DEVELOPER    │
                │  git commit /  │
                │     push       │
                └───────┬────────┘
                        │ (1) push code
                        ▼
                ┌────────────────────┐
                │  GITHUB REPOSITORY │
                │  Maven Web App     │
                │  (source + pom.xml)│
                └─────────┬──────────┘
                          │ (2) webhook / poll → checkout
                          ▼
   ┌──────────────────────────────────────────────────────────┐
   │  INSTANCE 1 — CI/CD SERVER (Jenkins)                       │
   │  + Maven (build)  + Ansible (config mgmt controller)       │
   │                                                            │
   │   Stage1 Checkout → Stage2 Sonar → Stage3 Gate            │
   │      → Stage4 Build WAR → Stage5 Ansible Deploy           │
   └───────┬───────────────────────────────────┬──────────────┘
           │ (3) send code for analysis         │ (5) ansible-playbook
           ▼                                    │     over SSH (WAR copy,
   ┌─────────────────────┐                      │     stop/start Tomcat)
   │ INSTANCE 2 —        │                      │
   │ CODE QUALITY SERVER │                      ▼
   │ (SonarQube)         │        ┌──────────────────────────────┐
   │ Bugs/Vulns/Smells   │        │ INSTANCE 3 — APPLICATION      │
   │ + QUALITY GATE      │        │ SERVER (Apache Tomcat)        │
   └──────────┬──────────┘        │ webapps/maven-web-app.war     │
              │ (4) gate result   │ → exploded → serves app       │
              │  PASS / FAIL      └───────────────┬───────────────┘
              ▼                                   │ (6) HTTP 200
   PASS → continue to Build                       ▼
   FAIL → STOP pipeline               http://<TOMCAT_IP>:8080/maven-web-app/
```

## Pipeline Workflow (decision flow)

```
Developer Commit
      ↓
Source Code Checkout            ← Jenkins clones GitHub repo, tracks commit
      ↓
Code Quality Validation         ← mvn sonar:sonar → SonarQube
      ↓
Quality Gate Decision  ──FAIL──► STOP (Tomcat untouched, old version stays live)
      ↓ PASS
Application Build               ← mvn clean package → target/*.war
      ↓
Artifact Generation             ← WAR archived in Jenkins (fingerprinted)
      ↓
Infrastructure Automation       ← Ansible: stop Tomcat, remove old WAR
      ↓
Application Deployment          ← Ansible: copy new WAR, start Tomcat
      ↓
Service Validation              ← Ansible uri check / curl → HTTP 200
```

## Tool integration flow

| From | To | How | Secured by |
|---|---|---|---|
| GitHub | Jenkins | git clone (HTTPS/SSH) | Jenkins credential store |
| Jenkins | SonarQube | `mvn sonar:sonar` + `withSonarQubeEnv` | `sonar-token` credential |
| SonarQube | Jenkins | `waitForQualityGate` (webhook) | server-to-server |
| Jenkins (Ansible) | Tomcat | SSH (`ansible-playbook`) | SSH private key, no passwords |

## Why reuse works

The architecture is **tool-identical** to the e-commerce pipeline — only the
*payload* (Maven WAR instead of the previous artifact) and three string values
(repo URL, Sonar project key, WAR name) differ. Same servers, same wiring.
