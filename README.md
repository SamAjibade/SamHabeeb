# Java Web App CI/CD Deployment with Jenkins, Tomcat, Maven, and Terraform on AWS

## Project Overview

This project demonstrates how I deployed a Java web application to an Apache Tomcat server using a Jenkins CI/CD pipeline and AWS infrastructure provisioned with Terraform.

The goal was to build an end-to-end DevOps workflow where Jenkins pulls source code from GitHub, builds the Java application with Maven, creates a WAR file, and deploys it to Apache Tomcat.

## Architecture

```text
GitHub
  ↓
Jenkins
  ↓
Maven Build
  ↓
WAR File
  ↓
Apache Tomcat
  ↓
Browser
```

## Tools Used

- AWS EC2
- Terraform
- Jenkins
- Apache Tomcat 9
- Maven
- Java
- Git and GitHub
- Ubuntu Linux
- VS Code

## Infrastructure Provisioned with Terraform

Terraform was used to create the AWS infrastructure for this project.

Resources created:

- VPC
- Public subnet
- Internet gateway
- Route table
- Route table association
- Security group
- Jenkins EC2 instance
- Tomcat EC2 instance

Inbound ports allowed in the security group:

| Port | Purpose |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 8080 | Jenkins and Tomcat |

## Jenkins Pipeline

The Jenkins pipeline contains four main stages:

1. Checkout SCM
2. Test
3. Build
4. Deploy to Tomcat

## Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'cd SampleWebApp && mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat-admin',
                        path: '',
                        url: 'http://TOMCAT_PUBLIC_IP:8080/'
                    )
                ],
                contextPath: 'webapp',
                war: 'SampleWebApp/target/SampleWebApp.war'
            }
        }
    }
}
```

## Tomcat Configuration

Tomcat was configured to allow Jenkins to deploy WAR files remotely.

The Tomcat users file was edited:

```bash
sudo vim /etc/tomcat9/tomcat-users.xml
```

The following roles and user were added:

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<user username="admin" password="example-password" roles="manager-gui,manager-script,admin-gui,admin-script"/>
```

Tomcat was restarted after the configuration change:

```bash
sudo systemctl restart tomcat9
```

## Problems I Faced and How I Solved Them

### 1. AWS Credentials Error

Terraform failed with this error:

```text
InvalidClientTokenId: The security token included in the request is invalid
```

I fixed it by reconfiguring my AWS credentials:

```bash
aws configure
aws sts get-caller-identity
```

### 2. EC2 Availability Zone Error

Terraform failed because the selected instance type was not supported in one Availability Zone.

I fixed it by choosing a supported Availability Zone:

```hcl
availability_zone = "us-east-1a"
```

### 3. SSH Key Permission Error

SSH failed because my private key permissions were too open.

I fixed the private key permissions on Windows using `icacls`.

### 4. Jenkins Installation Issue

Jenkins was not installed correctly at first.

I checked the Jenkins service:

```bash
sudo systemctl status jenkins
```

Then I installed Jenkins manually and started the service.

### 5. Jenkins GPG Key Error

While installing Jenkins, I got this error:

```text
NO_PUBKEY 7198F4B714ABFC68
```

I fixed it by using the newer Jenkins signing key.

### 6. Maven WAR Plugin Error

The Jenkins build failed because Maven was using an old WAR plugin:

```text
maven-war-plugin:2.2
Cannot access defaults field of Properties
```

I fixed it by adding a newer WAR plugin version in `pom.xml`:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <version>3.4.0</version>
</plugin>
```

### 7. Jenkinsfile Command Error

The original Jenkinsfile command was incorrect:

```bash
cd SampleWebApp mvn test
```

I fixed it by using `&&` to run the Maven command only after changing into the correct directory:

```bash
cd SampleWebApp && mvn test
```

### 8. Tomcat Deployment Credentials Error

Jenkins failed to deploy because Tomcat credentials were missing.

I fixed it by:

- Creating a Tomcat user with the `manager-script` role
- Adding Jenkins credentials with the ID `tomcat-admin`
- Updating the Jenkinsfile to use `credentialsId: 'tomcat-admin'`

## Screenshots

### Jenkins Dashboard

![Jenkins Dashboard](images/jenkins-dashboard.png)

### Failed Jenkins Builds During Troubleshooting

![Failed Jenkins Builds](images/jenkins-failed-builds.png)

### Successful Jenkins Pipeline

![Successful Jenkins Pipeline](images/jenkins-success-pipeline.png)

### Tomcat Homepage

![Tomcat Homepage](images/tomcat-homepage.png)

### Tomcat Manager

![Tomcat Manager](images/tomcat-manager.png)

### Deployed Java Web Application

![Deployed Java Web Application](images/tomcat-deployed-app.png)

## Final Result

The Jenkins pipeline successfully built and deployed the Java web application to Apache Tomcat.

Final application URL:

```text
http://TOMCAT_PUBLIC_IP:8080/webapp
```

## What I Learned

Through this project, I learned how to:

- Provision AWS infrastructure using Terraform
- Install and configure Jenkins
- Install and configure Apache Tomcat
- Build Java applications with Maven
- Deploy WAR files to Tomcat
- Use Jenkins credentials securely
- Debug Terraform, SSH, Jenkins, Maven, and Tomcat errors
- Document a DevOps project professionally

## Future Improvements

Possible improvements for this project:

- Use private subnets for better security
- Add an Application Load Balancer
- Use AWS Secrets Manager for credentials
- Add a GitHub webhook to trigger Jenkins automatically
- Use Jenkins agents instead of building on the controller
- Automate Jenkins and Tomcat configuration fully with Terraform user data

## Security Note

The public IP addresses and passwords shown in this project were used only for lab and learning purposes. In a real production environment, secrets should not be hardcoded or committed to GitHub.