# Alternative Method — Jenkins Freestyle Job (No Jenkinsfile)

If you prefer the UI / Freestyle approach instead of the declarative Jenkinsfile,
here's the equivalent. Same reused servers, same result.

> Pick ONE method — Pipeline (recommended, see [Jenkinsfile](Jenkinsfile)) **or** this Freestyle one.

---

## 1. Create the job
Jenkins → **New Item** → name `maven-web-app-freestyle` → **Freestyle project** → OK.

## 2. Source Code Management (Stage 1)
- Select **Git**.
- Repository URL: `<MAVEN_APP_REPO_URL>`
- Credentials: your existing GitHub credential (if private).
- Branch: `*/main` (or `*/master`).

## 3. Build Environment
- Tick **"Use secret text(s) or file(s)"** if you reference the Sonar token, or rely on
  the global SonarQube server config (reused).

## 4. Build Steps (in order)

### Step A — SonarQube analysis (Stage 2)
Add build step **"Execute SonarQube Scanner"** *(or "Invoke top-level Maven targets")*:

If using Maven goal:
```
clean compile sonar:sonar -Dsonar.projectKey=maven-web-app
```
*(Wrap with the SonarQube environment — set under "Build Environment → Prepare SonarQube
Scanner environment".)*

### Step B — Quality Gate (Stage 3)
Freestyle can't natively `waitForQualityGate`. Two options:
1. Install **"SonarQube Quality Gates"** / **"Quality Gates"** plugin and add the
   post-step check, **or**
2. Enable the SonarQube **webhook** + the "Skip the rest if Quality Gate fails" build step
   provided by the Sonar plugin.

> This is exactly why the **Pipeline method is recommended** — the gate check is one line there.

### Step C — Build the WAR (Stage 4)
Add **"Invoke top-level Maven targets"**:
- Maven Version: `maven3` (reused global tool)
- Goals: `clean package -DskipTests`

### Step D — Deploy via Ansible (Stage 5)
Add **"Execute shell"**:
```bash
cp target/*.war "$WORKSPACE/maven-web-app.war"
ansible-playbook -i hosts.txt deploy.yml \
  --extra-vars "war_src=$WORKSPACE/maven-web-app.war war_name=maven-web-app.war"
```
*(Put `hosts.txt` and `deploy.yml` in the job workspace or check them in alongside.)*

## 5. Post-build Actions
- **Archive the artifacts:** `maven-web-app.war`
- (Optional) Email notification on failure.

## 6. Save → Build Now → verify
- Console Output green.
- Browser: `http://<TOMCAT_IP>:8080/maven-web-app/`.

---

### Pipeline vs Freestyle — quick compare
| | Pipeline (Jenkinsfile) | Freestyle |
|---|---|---|
| Quality Gate stop | 1 line (`waitForQualityGate`) | needs extra plugin/webhook |
| Version controlled | ✅ in repo | ❌ UI only |
| Stage View | ✅ visual | ❌ single log |
| Recommended | ✅ | for quick demos |
