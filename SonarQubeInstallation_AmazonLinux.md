# SonarQube Installation

## Prerequisites
- SonarQube server will require 3GB+ RAM to work efficiently
### Install Database
```
rpm -ivh http://repo.mysql.com/mysql57-community-release-el7.rpm
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
yum install mysql-server -y
systemctl start mysqld
systemctl enable mysqld
grep 'temporary password' /var/log/mysqld.log
mysql_secure_installation
```
- create new password after login
### Install Java
```
yum install wget epel-release -y
yum install java -y
wget https://download.bell-sw.com/java/11.0.4/bellsoft-jdk11.0.4-linux-amd64.rpm
rpm -ivh bellsoft-jdk11.0.4-linux-amd64.rpm

```
- to change default java version, then select version (`bellsoft-jdk11.0.4`)
```
alternatives --config java
```
### Configure Linux System for Sonarqube
```
echo 'vm.max_map_count=262144' >/etc/sysctl.conf
sysctl -p
echo '* - nofile 80000' >>/etc/security/limits.conf
sed -i -e '/query_cache_size/ d' -e '$ a query_cache_size = 15M' /etc/my.cnf
systemctl restart mysqld
```
### Configure Database for Sonarqube
```
mysql -p -u root
mysql>
    create database sonarqube;
    create user 'sonarqube'@'localhost' identified by 'Redhat@123';
    grant all privileges on sonarqube.* to 'sonarqube'@'localhost';
    flush privileges;
```
### Install Sonarqube
```
yum install unzip -y
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.1.zip
cd /opt
unzip ~/sonarqube-7.9.1.zip
mv sonarqube-7.9.1 sonar
```
### Configure Sonarqube
```
sed -i -e '/^sonar.jdbc.username/ d' -e '/^sonar.jdbc.password/ d' -e '/^sonar.jdbc.url/ d' -e '/^sonar.web.host/ d' -e '/^sonar.web.port/ d' /opt/sonar/conf/sonar.properties
sed -i -e '/#sonar.jdbc.username/ a sonar.jdbc.username=sonarqube' -e '/#sonar.jdbc.password/ a sonar.jdbc.password=Redhat@123' -e '/InnoDB/ a sonar.jdbc.url=jdbc.mysql://localhost:3306/sonarqube?useUnicode=true&characterEncoding=utf&rewriteBatchedStatements=true&useConfigs=maxPerformance' -e '/#sonar.web.host/ a sonar.web.host=0.0.0.0' /opt/sonar/conf/sonar.properties
useradd sonar
chown sonar:sonar /opt/sonar/ -R
sed -i -e '/^#RUN_AS_USER/ c RUN_AS_USER=sonar' /opt/sonar/bin/linux-x86-64/sonar.sh
```
### Start SonarQube
```
/opt/sonar/bin/linux*/sonar.sh start
/opt/sonar/bin/linux*/sonar.sh status
/opt/sonar/logs
```


Note --> 

to aceess your sonarqube use openjdk17 java instead of bellsoft if your using latest sonarqube version

check /opt/sonarqube/conf/sonar.properties file is updated or not 
if not then update following attributes

sonar.jdbc.username=sonaruser
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc.mysql://localhost:3306/sonarqube
sonar.web.host=0.0.0.0
