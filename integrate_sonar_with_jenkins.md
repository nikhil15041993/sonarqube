# Integrate Sonarqube wit Jenkins 

## Prerequisites
1. a Sonarqube Server [Click here to Setup Sonarqube server]()
2. A Jenkins server [Click here to set up a Jenkins server]()

Integration Steps 

### On Sonarqube server 

1. Generate a sonarqube token to authenticate from Jenkins

### On Jenkins server 

1. Install Sonarqube plugin
1. Configure Sonarqube credentials (use secret text and paste the sonarqube token)
1. Add Sonarqube server with credentials and maven to jenkins "configure system" 
1. Install SonarScanner or install it on jenkins "configure system"
1. Run Pipeline job 


Jenkinsfile
===========

```
pipeline{
    agent any
    environment {
        PATH = "$PATH:/opt/apache-maven-3.8.2/bin"
    }
    stages{
       stage('GetCode'){
            steps{
                git 'https://github.com/nikhil15041993/hello-world.git'
            }
         }        
       stage('Build'){
            steps{
                sh 'mvn clean package'
            }
         }
        stage('SonarQube analysis') {
        // def scannerHome = tool 'SonarScanner 4.0';  ##desabled because we install automaticall on jenkins config system
        steps{
        withSonarQubeEnv('sonarqube-8.9') { 
        // If you have configured more than one global server connection, you can specify its name
        //  sh "${scannerHome}/bin/sonar-scanner"
        sh "mvn sonar:sonar"
    }
        }
        }
       
    }
}

```
