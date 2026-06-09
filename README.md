# MAVEN-APP-MILESTONE-3-GUIDE-

Show more
3:13 PM
Found it! The error is:

java.net.BindException: Address already in use
Port 9000 is already being used by another process.

Fix — Kill Whatever is Using Port 9000:
bash
fuser -k 9000/tcp
Then start SonarQube:

bash
sudo -u sonar /opt/sonarqube/bin/linux-x86-64/sonar.sh start
sleep 120
sudo -u sonar /opt/sonarqube/bin/linux-x86-64/sonar.sh status
If fuser doesn't work — find and kill manually:
bash
# Find what's using port 9000
lsof -i :9000

# Kill it (replace PID with actual number)
kill -9 <PID>
Then start SonarQube again.

Check what's on port 9000:
bash
ss -tlnp | grep 9000
Run fuser -k 9000/tcp first and tell me what you get! 🎯

