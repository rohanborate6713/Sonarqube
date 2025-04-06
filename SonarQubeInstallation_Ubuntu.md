# SonarQube Installation on Ubuntu

## 1. Configure Firewall


SonarQube web tool needs HTTP and HTTPS ports to work.

Open them using the Uncomplicated Firewall (UFW).

```
sudo ufw allow http
sudo ufw allow https
```
Check the firewall status.

```
sudo ufw status
```
## 2. Install OpenJDK
Install OpenJDK 11.

```
$ sudo apt install openjdk-11-jdk
```
## 3. Install PostgreSQL
Import the PostgreSQL repository key.

```
$ curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null
```
Add the PostgreSQL repository.

```
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
Update the system repository list.

```
sudo apt update
```
Install PostgreSQL 14.

```
sudo apt install postgresql postgresql-contrib
```
Check the status of the PostgreSQL service.

```
sudo systemctl status postgresql
```
## 4. Configure PostgreSQL
Log in to the PostgreSQL shell.

```
sudo -u postgres psql
```
Create the `sonaruser` role.

```
CREATE ROLE sonaruser WITH LOGIN ENCRYPTED PASSWORD 'your_password';
```
Create the `sonarqube` database.

```
CREATE DATABASE sonarqube;
```
Grant all privileges on the `sonarqube` database to the `sonaruser` role.

```
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonaruser;
```
Exit the shell.

```
 \q
```
Return to your default user account.

```
$ exit
```
## 5. Install Sonarqube
Copy the URL of the latest version of the community edition from the [﻿SonarQube downloads page](https://www.sonarqube.org/downloads/).

Download SonarQube using the URL copied above.

```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.6.1.59531.zip
```
Unzip the downloaded archive.

```
sudo apt install unzip
unzip -q sonarqube-9.6.1.59531.zip
```
Move the files to the `/opt/sonarqube` directory.

```
sudo mv sonarqube-9.6.1.59531 /opt/sonarqube
```
Delete the downloaded archive.

```
rm sonarqube-9.6.1.59531.zip
```
## 6. Create SonarQube User
Create a system user along with the group for SonarQube.

```
sudo adduser --system --no-create-home --group --disabled-login sonarqube
```
Give Sonar user permissions to the `/opt/sonarqube` directory.

```
sudo chown sonarqube:sonarqube /opt/sonarqube -R
```
## 7. Configure SonarQube Server
Open the SonarQube configuration file for editing.

```
sudo nano /opt/sonarqube/conf/sonar.properties
```
Find the following lines.

```
#sonar.jdbc.username=
#sonar.jdbc.password=
```
Uncomment them by removing the hash in front of them and adding the database credentials created in step 4.

```
sonar.jdbc.username=sonaruser
sonar.jdbc.password=your_password
```
Find the following line.

```
#sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube?currentSchema=my_schema
```
Uncomment it and replace the existing value with the following.

```
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```
Find the following lines.

```
#sonar.web.javaAdditionalOpts=-server
#sonar.web.host=0.0.0.0
```
Configure the following settings, so SonarQube listens to localhost only because Nginx handles the external connections.

```
sonar.web.javaAdditionalOpts=-server
sonar.web.host=127.0.0.1

if you want to access sonarqube on ec2 public IP then 
sonar.web.host=0.0.0.0
```
Save the file by pressing Ctrl+X, then Y.

Increase the virtual memory on the system for Elasticsearch to function. Open the `sysctl.conf` file for editing.

```
sudo nano /etc/sysctl.conf
```
Paste the following lines at the end of the file.

```
vm.max_map_count=524288
fs.file-max=131072
```
Save the file by pressing Ctrl+X, then Y.

Create the file `/etc/security/limits.d/99-sonarqube.conf` and open it for editing.

```
sudo nano /etc/security/limits.d/99-sonarqube.conf
```
Paste the following lines to increase the file descriptors and threads that the `sonarqube` user can open.

```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```
Save the file by pressing Ctrl+X, then Y.

Reboot the system to apply the changes.

```
sudo reboot
```
## 8. Setup Sonar Service
Create the systemd service file for Sonar and open it for editing.

```
sudo nano /etc/systemd/system/sonarqube.service
```
Paste the following code in it.

```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
Restart=always

StandardOutput=syslog
LimitNOFILE=131072
LimitNPROC=8192
TimeoutStartSec=5
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```
Save the file by pressing Ctrl+X, then Y.

Start the SonarQube service.

```
sudo systemctl start sonarqube
```
Check the status of the service.

```
sudo systemctl status sonarqube
```
Enable the service to start automatically at boot.

```
sudo systemctl enable sonarqube
```
Verify if the Sonarqube server is functioning properly.

```
curl http://127.0.0.1:9000
or
http:<your-server-publicIP>:9000
```
Look for the following text in the HTML response. for `curl http://127.0.0.1:9000` 

```
<script>
    window.baseUrl = '';
    window.serverStatus = 'UP';
    window.instance = 'SonarQube';
    window.official = true;
</script>
```
This confirms everything is working fine.

