# Step-by-Step Commands — Maven App CI/CD (Reusing Existing Servers)

Follow top to bottom. **You are NOT building new servers** — only verifying the
existing ones, creating a new Jenkins job + Sonar project, and running the pipeline.

> Legend — replace these everywhere:
> `<JENKINS_IP>` `<SONAR_IP>` `<TOMCAT_IP>` `<MAVEN_APP_REPO_URL>`

---

## PART 0 — Confirm the 3 existing servers are alive (pre-flight)

SSH into the **Jenkins** box (Instance 1):

```bash
ssh -i your-key.pem ubuntu@<JENKINS_IP>

sudo systemctl status jenkins        # active (running)
java -version
mvn -version
ansible --version
git --version
```

Check Sonar + Tomcat reachable from the Jenkins box:

```bash
curl -I http://<SONAR_IP>:9000       # expect 200
curl -I http://<TOMCAT_IP>:8080      # expect 200
```

✅ All good → continue. ❌ A server down → just start it (don't recreate):
```bash
sudo systemctl start jenkins         # on Jenkins box
sudo systemctl start sonarqube       # on Sonar box (or its start script)
sudo systemctl start tomcat          # on Tomcat box
```

---

## PART 1 — Source Code Management (Stage 1)

You have two repos:
1. **Maven application repo** (instructor-provided) — the code that gets built.
2. **This guide repo** — holds Jenkinsfile, deploy.yml, hosts.txt.

```bash
# Test the Maven app repo clones fine
git clone <MAVEN_APP_REPO_URL>
cd maven-web-application
git log -1 --oneline
git branch -a
```

> If the Maven repo is private, add a GitHub credential in Jenkins (reuse your existing one).

---

## PART 2 — Static Code Analysis + Quality Gate (Stages 2 & 3)

SonarQube is already installed (Instance 2). You only add a **new project**.

1. Open `http://<SONAR_IP>:9000` → log in.
2. **Quality Gates** → confirm a gate exists (e.g. "Sonar way") and is set as default.
   - Conditions usually: Bugs, Vulnerabilities, Code Smells, Coverage, Duplications.
3. You can **reuse the existing token** (Jenkins credential `sonar-token`).
   To make a fresh one: My Account → Security → Generate Token → copy.
4. Confirm in Jenkins: **Manage Jenkins → System → SonarQube servers**
   has your server named `sonar-server` with that token. (Already there from e-commerce.)

Manual test of analysis from the app folder (optional sanity check):

```bash
cd maven-web-application
mvn clean compile sonar:sonar \
  -Dsonar.projectKey=maven-web-app \
  -Dsonar.host.url=http://<SONAR_IP>:9000 \
  -Dsonar.login=<SONAR_TOKEN>
```

Open Sonar UI → project `maven-web-app` → see Bugs / Vulnerabilities / Code Smells / Gate status.

---

## PART 3 — Application Build (Stage 4)

```bash
cd maven-web-application
mvn clean package -DskipTests
ls -lh target/*.war          # <-- your deployable WAR artifact
```

---

## PART 4 — Configuration Management + Deploy (Stage 5, Ansible → Tomcat)

On the Jenkins box, put `hosts.txt` and `deploy.yml` in the workspace (they ship in this repo).

```bash
# 1) Edit hosts.txt -> set <TOMCAT_SERVER_IP>, ssh user, key path
# 2) Confirm Ansible can reach Tomcat (reuses your existing SSH key)
ansible -i hosts.txt tomcat -m ping        # expect SUCCESS / "pong"

# 3) Dry-run the deploy playbook
ansible-playbook -i hosts.txt deploy.yml \
  --extra-vars "war_src=$(pwd)/target/*.war war_name=maven-web-app.war" --check

# 4) Real deploy (Jenkins will run this automatically)
ansible-playbook -i hosts.txt deploy.yml \
  --extra-vars "war_src=$(pwd)/maven-web-app.war war_name=maven-web-app.war"
```

Verify:
```bash
curl -I http://<TOMCAT_IP>:8080/maven-web-app/      # expect HTTP 200
```

---

## PART 5 — Wire it all into a Jenkins Pipeline job

1. Jenkins UI → **New Item** → name `maven-web-app-pipeline` → **Pipeline** → OK.
2. Scroll to **Pipeline** section → Definition: **Pipeline script from SCM**.
   - SCM: **Git**
   - Repository URL: this guide repo URL (where Jenkinsfile lives)
   - Branch: `*/master` (or `*/main`)
   - Script Path: `Jenkinsfile`
   *(Alternatively choose "Pipeline script" and paste the Jenkinsfile contents directly.)*
3. **Save** → **Build Now**.
4. Open **Console Output** / **Stage View** — all 5 stages should go green:
   `Checkout → SonarQube → Quality Gate → Build WAR → Deploy via Ansible`.

> Edit the 4 EDIT-THESE values at the top of [Jenkinsfile](Jenkinsfile) first:
> `APP_REPO`, `APP_BRANCH`, `SONAR_KEY`, `WAR_NAME`.

---

## PART 6 — Test the Quality Gate FAIL path (proves Stage 3 works)

To demonstrate the pipeline stops on bad code:
1. In SonarQube, tighten the Quality Gate (e.g. "0 bugs allowed" / coverage 80%).
2. Re-run the build. If the app violates it, the **Quality Gate stage fails** and
   the pipeline **never reaches Build/Deploy** → Tomcat keeps the old version.
3. Screenshot this (see [SCREENSHOTS-GUIDE.md](SCREENSHOTS-GUIDE.md)). Then revert the gate.

---

## PART 7 — Git repo for this guide (push your work)

See the bottom of [completeFlow.txt](completeFlow.txt) for the exact `git init / push` commands.
