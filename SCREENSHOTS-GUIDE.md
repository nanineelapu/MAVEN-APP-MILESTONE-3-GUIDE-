# Screenshots to Capture — Maven App CI/CD (Submission Evidence)

Capture these in order. Each maps to a case-study requirement, so graders can tick them off.

| # | Screenshot | Where | Proves |
|---|---|---|---|
| 1 | 3 running instances (Jenkins/Sonar/Tomcat) | Cloud console / `systemctl status` | Infra reused, not rebuilt |
| 2 | Maven app repo on GitHub (with pom.xml) | GitHub | Source code management |
| 3 | Jenkins job config — "Pipeline script from SCM" | Jenkins → job → Configure | Stage 1 wiring |
| 4 | Jenkins **Stage View** all 5 stages green | Jenkins → job | End-to-end pipeline success |
| 5 | Console Output — `git log -1` / checkout success | Build → Console | Stage 1 done + version tracking |
| 6 | Console Output — SonarQube analysis lines | Build → Console | Stage 2 done |
| 7 | SonarQube dashboard — project `maven-web-app` (Bugs/Vulns/Smells) | `http://<SONAR_IP>:9000` | Static analysis results |
| 8 | SonarQube **Quality Gate = Passed** badge | Sonar project page | Quality Gate PASS |
| 9 | Console Output — `target/*.war` built | Build → Console | Stage 4 WAR artifact |
| 10 | Jenkins **archived artifact** (the .war) | Build → Artifacts | Artifact generation |
| 11 | Console Output — `ansible-playbook` tasks ok | Build → Console | Stage 5 / config mgmt |
| 12 | `ansible -m ping` SUCCESS | Terminal | Ansible↔Tomcat connectivity |
| 13 | Tomcat webapps showing the new WAR | `ls /opt/tomcat/webapps` | Deployment replaced old |
| 14 | **App running in browser** `http://<TOMCAT_IP>:8080/maven-web-app/` | Browser | Service validation |
| 15 | (Optional) Quality Gate **FAIL** stopping the pipeline | Jenkins stage view (red) | Failure-handling proof |
| 16 | Jenkins **Build History** (multiple builds) | Jenkins → job | Build history maintained |

## Tips
- For #15: temporarily tighten the Quality Gate in SonarQube, rebuild, screenshot the
  red "Quality Gate" stage + the "Pipeline aborted" message, then revert the gate.
- Put screenshots in a `screenshots/` folder and reference them in your report.
- Name them `01-instances.png`, `02-repo.png`, … so order is obvious.
