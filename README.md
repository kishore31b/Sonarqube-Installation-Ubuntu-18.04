# Sonarqube-Installation-Ubuntu-18.04

Source: https://www.digitalocean.com/community/tutorials/how-to-ensure-code-quality-with-sonarqube-on-ubuntu-18-04



JAVA-8 Installation
====================

sudo apt update
sudo apt install openjdk-8-jdk
java -version
sudo update-alternatives --config java
sudo nano /etc/environment
sudo cat /etc/environment
		PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
		JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64/jre/bin"
source /etc/environment
echo $JAVA_HOME
		/usr/lib/jvm/java-8-openjdk-amd64/jre/bin
==================================================================================================================

Installing Nginx Server
=======================

sudo apt update
sudo apt install nginx
curl http://13.235.103.219/

===============================================================================================================

Installing MySQL to Manage Site Data
====================================

sudo apt install mysql-server
sudo mysql_secure_installation
y		// for all
n 		// for Deny Root Login
sudo mysql
	SELECT user,authentication_string,plugin,host FROM mysql.user;
	ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
	FLUSH PRIVILEGES;
	SELECT user,authentication_string,plugin,host FROM mysql.user;
	exit

====================================================================================================================

Installing Sonarqube
====================

sudo adduser --system --no-create-home --group --disabled-login sonarqube
sudo mkdir /opt/sonarqube
sudo apt-get install unzip
mysql -u root -p	
	CREATE DATABASE sonarqube;
	CREATE USER sonarqube@'localhost' IDENTIFIED BY 'some_secure_password';
	GRANT ALL ON sonarqube.* to sonarqube@'localhost';
	FLUSH PRIVILEGES;
	EXIT;
cd /opt/sonarqube
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.5.zip
sudo unzip sonarqube-7.5.zip
sudo rm sonarqube-7.5.zip
sudo chown -R sonarqube:sonarqube /opt/sonarqube
sudo nano sonarqube-7.5/conf/sonar.properties
		sonar.jdbc.username=sonarqube
    	sonar.jdbc.password=some_secure_password
   		sonar.jdbc.url=jdbc:mysql://localhost:3306/sonarqube?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs	=maxPerformance&useSSL=false
   		sonar.web.javaAdditionalOpts=-server
	    sonar.web.host=127.0.0.1
sudo nano /etc/systemd/system/sonarqube.service
--------------------------------------------------------------------------
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/sonarqube-7.5/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/sonarqube-7.5/bin/linux-x86-64/sonar.sh stop

User=sonarqube
Group=sonarqube
Restart=always

[Install]
WantedBy=multi-user.target
---------------------------------------------------------------------------
sudo service sonarqube start
service sonarqube status
sudo systemctl enable sonarqube
sudo netstat -plnt
		Check 9000 is listening or not, if not then give next command
sudo sysctl -w vm.max_map_count=262144
sudo service sonarqube restart
sudo netstat -plnt
curl http://127.0.0.1:9000
		You will get some result
sudo nano /etc/nginx/sites-enabled/sonarqube
---------------------------------------------------------------------------
server {
    listen 80;
    server_name 13.235.103.219;

    location / {
        proxy_pass http://127.0.0.1:9000;
    }
}
--------------------------------------------------------------------------------
sudo nginx -t
sudo service nginx restart
curl http://13.235.103.219
------------------------------------------------------------------------------------

To change Port of Nginx and Sonar to run in nginx
-------------------------------------------------

cd /etc/nginx/sites-enabled/ 
sudo vi default
		change to your wish in Listen : 7070
sudo nano /etc/nginx/sites-enabled/sonarqube
---------------------------------------------------------------------------
server {
    listen 7070;
    server_name 13.235.103.219;

    location / {
        proxy_pass http://127.0.0.1:9000;
    }
}
--------------------------------------------------------------------------------
sudo nginx -t
sudo service nginx restart
curl http://13.235.103.219:7070


-------------------------------------------------------------------------

To check errors in Sonarqube
===============================
cd /opt/sonarqube/sonarqube-7.5/logs
vi sonar.log

===========================================================================
