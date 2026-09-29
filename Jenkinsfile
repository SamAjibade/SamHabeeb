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
                        url: 'http://100.53.45.221:8080/'
                    )
                ],
                contextPath: 'webapp',
                war: 'SampleWebApp/target/SampleWebApp.war'
            }
        }
    }
}