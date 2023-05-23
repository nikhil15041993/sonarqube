## Sonarqube System Requirements

Following are the minimum server requirement for running the Sonarqube server.

* Server with minimum 2GB RAM and 1 vCPU capacity
* PostgreSQL version 9.3 or greater.
* OpenJDK 11 or JRE 11
* All Sonarquber processes should run as a non-root sonar user.


## Setup PostgreSQL 10 For SonarQube


### Step 1: Install PostgreSQL 10 repo.

```
sudo yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm -y
```
### Step 2: Install PostgreSQL 10

```
sudo yum install postgresql10-server postgresql10-contrib -y
```

### Step 3: Initialize the database.

```
sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```

### Step 4: Open /var/lib/pgsql/data/pg_hba.conf file to change the authentication to md5.

```
sudo vi /var/lib/pgsql/10/data/pg_hba.conf
```

Find the following lines at the bottom of the file and change peer to trust and idnet to md5

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```
Once changed, it should look like the following.

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

### Step 5: Start and enable PostgreSQL.

```
sudo systemctl start postgresql-10
sudo systemctl enable postgresql-10
```

### Step 6: You can verify the installation using the following version select query.

```
sudo -u postgres /usr/pgsql-10/bin/psql -c "SELECT version();"
```


## Create Sonar User and Database

We need to have a sonar user and database for the sonar application.

Step 1: Change the default password of the Postgres user. All Postgres commands have to be executed by this user.

```
sudo passwd postgres
```

### Step 2: Login as postgres user with the new password.

```
su - postgres
```

### Step 3: Log in to the PostgreSQL CLI.

```
psql
```

### Step 4: Create a sonarqubedb database.

```
create database sonarqubedb;
```

### Step 5: Create the sonarqube DB user with a strongly encrypted password. Replace your-strong-password with a strong password.

```
create user sonarqube with encrypted password 'your-strong-password';
```

### Step 6: Next, grant all privileges to sonrqube user on sonarqubedb.

```
grant all privileges on database sonarqubedb to sonarqube
```

### Step 7: Exit the psql prompt using the following command.

```
\q
```

### Step 6: Switch to your sudo user using the exit command.

```
exit
```


## Setup Sonarqube Web Server

### Step 1: Download the latest Sonarqube installation file to /opt folder. You can get the latest download link from here.

```
cd /opt 
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.6.zip
```

### 2. Unzip sonarqube source files and rename the folder.

```
sudo unzip sonarqube-7.6.zip
sudo mv sonarqube-7.6 sonarqube
```

### 4. Open /opt/sonarqube/conf/sonar.properties file.

```
sudo vi /opt/sonarqube/conf/sonar.properties
```
Uncomment and edit the parameters as shown below. Change the password accordingly. You will find the JDBC parameter under the PostgreSQL section.

```
sonar.jdbc.username=sonar                                                                                                                     
sonar.jdbc.password=sonar-db-password
sonar.jdbc.url=jdbc:postgresql://localhost/onarqubedb
```
By default, sonar will run on 9000. If you want on port 80 or any other port, change the following parameters for accessing the web console on that specific port.

```
sonar.web.host=0.0.0.0
sonar.web.port=80
```
If you want to access Sonarqube some path like http://url:/sonar, change the following parameter.

```
sonar.web.context=/sonar
```


## Add Sonar User and Privileges
Create a user named sonar and make it the owner of the /opt/sonarqube directory.

```
sudo useradd sonar
sudo chown -R sonar:sonar /opt/sonarqube
```

## Start Sonarqube Service
To start sonar service, you need to use the script in sonarqube bin directory.


### Step 1: Login as sonar user

```
sudo su - sonar
```

### Step 2: Navigate to the start script directory.

```
cd /opt/sonarqube/bin/linux-x86-64 
```

### Step 3: Start the sonarqube service.

```
./sonar.sh start
```
Now, you should be able to access sonarqube on the browser on port 9000

### Step 4: Check the application status. If it is in a running state, you can access the sonarqube dashboard using the DNS name or Ip address of your server.

```
sudo ./sonar.sh status
```


## Setting up Sonarqube as Systemd Service

### Step 1: Create a file /etc/systemd/system/sonarqube.service

```
sudo vi /etc/systemd/system/sonarqube.service
```

### Step 2: Copy the following content onto the file.

```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=simple
User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
ExecStart=/bin/nohup java -Xms32m -Xmx32m -Djava.net.preferIPv4Stack=true -jar /opt/sonarqube/lib/sonar-application-7.6.jar
StandardOutput=syslog
LimitNOFILE=65536
LimitNPROC=8192
TimeoutStartSec=5
Restart=always

[Install]
WantedBy=multi-user.target
```

### Step 3: Start and enable sonarqube

```
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
```

### Step 4: Check the sonarqube status to ensure it is running as expected.

```
sudo systemctl status  sonarqube
```

## Troubleshooting Sonarqube
All the logs of sonarqube are present in the /opt/sonarqube/logs directory.

```
cd /opt/sonarqube/logs
```
You can find the following log files.

```
es.log
sonar.log
web.log
access.log
```
Using the tail command you can check the latest logs. For example,

```
tail -f access.log
```
