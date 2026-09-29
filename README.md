# realcloud\_java-new\_project

\# Java Web App CI/CD Deployment with Jenkins, Tomcat, Maven, and Terraform on AWS



\## Project Overview



This project demonstrates how I deployed a Java web application to an Apache Tomcat server using Jenkins CI/CD automation and Terraform-provisioned AWS infrastructure.



The goal was to build an end-to-end DevOps pipeline where Jenkins pulls source code from GitHub, builds the Java application with Maven, creates a WAR file, and deploys it to Apache Tomcat.



\## Architecture



```text

GitHub

&#x20; ↓

Jenkins

&#x20; ↓

Maven Build

&#x20; ↓

WAR File

&#x20; ↓

Apache Tomcat

&#x20; ↓

Browser











Tools Used

\- AWS EC2

\- Terraform

\- Jenkins

\- Apache Tomcat 9

\- Maven

\- Java

\- Git and GitHub

\- Ubuntu Linux

\- VS Code

Infrastructure with Terraform

Terraform was used to create the AWS infrastructure, including:

\- VPC

\- Public subnet

\- Internet gateway

\- Route table

\- Route table association

\- Security group

\- Jenkins EC2 instance

\- Tomcat EC2 instance

The security group allowed inbound traffic on:

22    SSH

80    HTTP

443   HTTPS

8080  Jenkins and Tomcat





Jenkins Pipeline

The Jenkins pipeline contains the following stages:

1\. Checkout SCM

2\. Test

3\. Build

4\. Deploy to Tomcat





The final Jenkinsfile:





pipeline {

&#x20;   agent any



&#x20;   stages {

&#x20;       stage('Test') {

&#x20;           steps {

&#x20;               sh 'cd SampleWebApp \&\& mvn test'

&#x20;           }

&#x20;       }



&#x20;       stage('Build') {

&#x20;           steps {

&#x20;               sh 'cd SampleWebApp \&\& mvn clean package'

&#x20;           }

&#x20;       }



&#x20;       stage('Deploy to Tomcat') {

&#x20;           steps {

&#x20;               deploy adapters: \[

&#x20;                   tomcat9(

&#x20;                       credentialsId: 'tomcat-admin',

&#x20;                       path: '',

&#x20;                       url: 'http://TOMCAT\_PUBLIC\_IP:8080/'

&#x20;                   )

&#x20;               ],

&#x20;               contextPath: 'webapp',

&#x20;               war: 'SampleWebApp/target/SampleWebApp.war'

&#x20;           }

&#x20;       }

&#x20;   }

}





Tomcat Configuration

Tomcat was configured to allow Jenkins to deploy WAR files remotely.

The Tomcat users file was edited:

sudo vim /etc/tomcat9/tomcat-users.xml



The following roles and user were added:

<role rolename="manager-gui"/>

<role rolename="manager-script"/>

<role rolename="admin-gui"/>

<role rolename="admin-script"/>

<user username="admin" password="example-password" roles="manager-gui,manager-script,admin-gui,admin-script"/>





Tomcat was restarted:

sudo systemctl restart tomcat9





Problems I Faced and How I Fixed Them:



1\. AWS Credentials Error

Terraform failed with:

InvalidClientTokenId: The security token included in the request is invalid

I fixed it by reconfiguring AWS credentials:

aws configure

aws sts get-caller-identity



2\. EC2 Availability Zone Error:



Terraform failed because the instance type was not supported in one Availability Zone.

I fixed it by choosing a supported Availability Zone:

availability\_zone = "us-east-1a"



3\. SSH Key Permission Error:



SSH failed because my private key permissions were too open.

I fixed the Windows key permission using icacls.



4\. Jenkins Installation Issue:



Jenkins was not installed correctly at first. I checked the service:

sudo systemctl status jenkins

Then installed Jenkins manually and started the service.





5\. Jenkins GPG Key Error:



While installing Jenkins, I got:

NO\_PUBKEY 7198F4B714ABFC68

I fixed it by using the newer Jenkins signing key.





6\. Maven WAR Plugin Error:



The Jenkins build failed because Maven used an old WAR plugin:

maven-war-plugin:2.2

Cannot access defaults field of Properties

I fixed it by adding a newer plugin version in pom.xml:

<plugin>

&#x20; <groupId>org.apache.maven.plugins</groupId>

&#x20; <artifactId>maven-war-plugin</artifactId>

&#x20; <version>3.4.0</version>

</plugin>





7\. Jenkinsfile Command Error:



The original command was wrong:

cd SampleWebApp mvn test





I fixed it with:

cd SampleWebApp \&\& mvn test



8\. Tomcat Deployment Credentials Error:



Jenkins failed to deploy because Tomcat credentials were missing.

I fixed it by:

\- Creating a Tomcat user with manager-script

\- Adding Jenkins credentials with ID tomcat-admin

\- Updating the Jenkinsfile to use credentialsId: 'tomcat-admin'









Final Result

The pipeline successfully deployed the Java web application to Apache Tomcat.

Final application URL:

http://TOMCAT\_PUBLIC\_IP:8080/webapp





What I Learned

Through this project, I learned how to:

\- Provision AWS infrastructure using Terraform

\- Install and configure Jenkins

\- Install and configure Apache Tomcat

\- Build Java applications with Maven

\- Deploy WAR files to Tomcat

\- Use Jenkins credentials

\- Debug CI/CD pipeline errors

\- Document DevOps projects professionally



