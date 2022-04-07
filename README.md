# Sonarqube Setup

SonarQube is an open-source static testing analysis software, it is used by developers to manage source code quality and consistency.

## Prerequisites

Source: https://docs.sonarqube.org/latest/requirements/requirements/
1. An EC2 instance with a minimum of 2 GB RAM (t2.small)  
2. Java 11 installation 

SonarQube cannot be run as root on Unix-based systems, so create a dedicated user account for SonarQube if necessary.

## Installation steps

1. Download SonarQube [latest verions](https://www.sonarqube.org/downloads/) on to EC2 instace 
   
   ```
   cd /opt  
   wget  https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.8.54436.zip  
   ```
   
   extract packages
   ``` 
   unzip /opt/sonarqube-8.9.8.54436.zip 
   ```
   
2. Change ownershipt to the user and Switch to Linux binaries directory to start service

   ```
    chown -R <sonar_user>:<sonar_user_group> /opt/sonarqube-x.x  
    cd /opt/sonarqube-x.x/bin/linux-x86-64   
    ./sonar.sh start
   ```
   
   
3. Connect to the SonarQube server through the browser. It uses port 9000. 

  ```
   http://<Public-IP>:9000
   ```
   
 ## 🧹 CleanUp  
   1. Stop SonarQube server
 
 ``` 
   cd /opt/sonarqube-x.x/bin/linux-x86-64 
   ./sonar.sh stop
   ```
   
