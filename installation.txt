Jekins Installation

#! /bin/bash

sudo apt update -y 
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update -y
sudo apt install maven
sudo apt install jenkins -y
sudo systemctl start jenkins
echo 'clearing screen...' && sleep 5
clear
echo 'Jenkins is Installed'
echo 'This is the Default password Now :' $(sudo cat /var/lib/jenkins/secrets/initialAdminPassword)



#Install Tomcat9

sudo apt update -y
sudo apt-get install tomcat9 tomcat9-docs tomcat9-admin -y
sudo cp -r /usr/share/tomcat9-admin/* /var/lib/tomcat9/webapps/ -v 
sudo vim /var/lib/tomcat9/conf/tomcat-users.xml

enter this code
<role rolename="manager-script"/>
<user username="tomcat" password="password" roles="manager-script"/>
<role rolename="admin-gui"/>
<role rolename="manager-gui"/>
<user username="admin" password="admin" roles="admin-gui,manager-gui"/>

sudo systemctl restart tomcat9
sudo systemctl status tomcat9
