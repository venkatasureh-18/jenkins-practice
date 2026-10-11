JENKINS: 
ITS A FREE AND OPEN-SOURCE TOOL.
IT IS PLATFORM INDEPENDENT.
IT CAN AUTOMATE ENTIRE SDLC. 
PORT NUMBER: 8080
JAVA: JAVA-17
DEFAULT PATH: /var/lib/Jenkins



JENKINS WRITTEN ON JAVA.
IT CONSIST OF PLUGINS.
WE HAVE COMMUNITY SUPPORT.
IT IS OWNED BY SUN MICRO SYSTEM AS HUDSON.
HUDSON IS PAID VERSION.
LATER ORACLE BROUGHT HUDSON AND MAKE IT FREE.
LATER HUDSON WAS RENAMED AS JENKINS.
INVENTOR: Kohsuke Kawaguchi


ALTERNATIVES:
GITLAB, BAMBOO, GO CI, CIRCLE CI, TARVIS, SEMAPHORE, BUDDY BUILD MASTER, HARNESS, ARGOCD -----

CLOUD: AWS CODEPIPELINE, AZURE PIPLEINE ---------------------


SETUP: Create an EC2 with UBUNTU 24.04 AMI with c7i.flex-large & 8080 in sg


sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins


CONNECT:
copy-public-ip:8080 (browser)
cat /var/lib/jenkins/secrets/initialAdminPassword (server)
paster password on browser -- > installing plugins --- > user details -- > save --> save -- > start using jenkins

HOW TO INSTALL MAVEN TOOL:
MAVEN IS A BUILD TOOL
THAT IS RESPONSIBLE FOR BUILD TEST AND CONVERT ARTIFACT (WAR)

MANAGE JENKINS (SETTINGS SYMBOL) -- > TOOLS -- > ADD MAVEN -- > name: maven -- > SAVE



JOB: it is used to perform task.
to do any work or task in jenkins we need to create a job.

to run commands we need to select execute shell on build steps.

build now: to run job
workspace: place where our job outputs will store

CREATING A JOB:
NEW ITEM -- > NAME: ci-job -- > FREESTYLE -- > OK -- > SCM -- > GIT -- > REPOURL: https://github.com/devopsbyraham/jenkins-java-project.git -- >Build Steps -- > ADD Build Steps -- > Invoke top-level Maven targets -- > Maven Version: maven -->  clean package -- > save -- > build now

WORKSPACE: where your job output is going to be stored
Default: /var/lib/jenkins/workspace


CUSTOM WORKSPACE: 
GO TO JOB -- > CONFIGURE -- > Advance Options -- > Use custom workspace -- > /root/raham

cd 
mkdir raham
chown jenkins:jenkins /root
chown jenkins:jenkins /root/raham

Now we can see output on root folder after building.

CUSTOM PORT NUMBER:
IF A TOOL IS HAVING A PORT NUMBER WE NEED TO CALL IT AS SERVICE
FOR EVERY SERVICE WE HAVE A CONFIGURATION FILE.
TO CHANGE ANY PROPERTY OF A SERVICE WE NEED TO MODIFY THE CONFIG FILE.

find / -name jenkins.service

vim /usr/lib/systemd/system/jenkins.service
LINE 72: 8080=8090
systemctl daemon-reload
systemctl restart jenkins

=============================================================================


INSTALLING MAVEN ON JENKINS
Dashboard
Manage Jenkins
Tools
Maven installations
name: maven
save



CRON JOB: We can schedule the jobs that need to be run at particular intervals.
here we use cron syntax
cron syntax has * * * * *
each * is separated by space

*	: minutes
*	: hours
*	: date
*	: month
*	: day of week (sun=0, mon=1 ----)


6:32 PM 15 JULY 2025

32 18 15 7 2

create a ci job -- > Build Triggers -- > Build periodically -- > * * * * * -- > save

CRONTAB-GENERATOR: https://crontab-generator.org/

limitation: it will not check the code is changed or not.



POLL SCM: 
in pollscm we will set time limit for the jobs.
if dev commit the code it will wait until the time is done.
in given time if we have any changes on code it will generate a build

create a ci job -- > Build Triggers -- > poll scm -- > * * * * * -- > save
commit the changes in GitHub then wait for 1 min.

LIMITATION:
1. in pollscm, we need to wait for the time we set.
2. we will get the last commit only.

WEBHOOK: it will trigger build the moment we change the code.
here we need not to wait for the build.

create ci job -- > Build Triggers: GitHub hook trigger for GITScm polling -- > save

repo on github -- > settings -- > webhooks -- > add webhook -- > Payload URL (jenkins url) -- > http://35.180.46.134:8080/github-webhook/  -- > Content type -- > application/json -- > add



BUILD SCRIPTS: to make jenkins builds from remote loc using script/
give token 
give url on other browser.
create ci job -- > Build Triggers: Trigger builds remotely -- > token -- > save



POINT-1:
Jenkins will execute builds sequentially.
by default jenkins will not do concurrent builds.
we need to enable this option in configuration.

Execute concurrent builds if necessary -- > tick it

POINT-2:
MAX BY DEFAULT JENKINS CAN DO 2 BUILDS
IF I WANT MAKE 4 BUILDS AT A TIME


Jenkins -- > Build Executor Status -- > 
Built-In Node
Configure
Number of executors :4
save 



THROTTLE BUILD:

To restrict the builds in a certain time or intervals.
if we dont restrict due to immediate builds jenkins might crashdown.

 
create a ci job -- > configure -- > Throttle builds -- > Number of builds: 3 -- > time period : hours -- > save

now it will take 20 minutes of gap for each build.

IF JENKINS GOT CRASHED
STOP THE SERVER AND START IT AFTER 2 MINS
	
NOTE: chkconfig jenkins on
it will restart Jenkins automatically




CUSTOM THEMES:

dashboard -- > manage Jenkins  -- > PLUGINS -- > Simple Them & Customizable Header -- > SELECT AND INSTALL ALL OF THEM -- >  
manage Jenkins -- > Appearance -- Customizable theme -- > ADD & CSS
https://github.com/alefnode/jenkins-themes (TO GET DIFFERENT THEMS)

https://cdn.rawgit.com/afonsof/jenkins-material-theme/gh-pages/dist/material-cyan.css

ADD HEADERS OF NARESHIT AND USE LOGO FROM IMAGES.

==========================================================================================

MASTER AND SLAVE:
it is used to distribute the builds.
it reduce the load on jenkins server.
communication blw master and slave is ssh.
Here we need to install agent (java).
SLAVE IS your instance.
slave can use any platform.
label = way of assigning work for slave.


SETUP:
#STEP-1 : Create a server and install  JAVA-1.8.0 maven
sudo apt update
sudo apt install -y wget gnupg
wget -qO - https://repos.azul.com/azul-repo.key   sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/azul.gpg
echo "deb https://repos.azul.com/zulu/deb stable main"   sudo tee /etc/apt/sources.list.d/zulu.list
sudo apt update
sudo apt install -y zulu8-jdk
sudo apt install -y maven


#DONT RUN BELOW COMMANDS WHEN YOU GOT BUILD FAILED
sudo apt install -y openjdk-17-jdk
update-alternatives --config java
set java to version17

#STEP-2: SETUP THE SLAVE SERVER
Dashboard -- > Manage Jenkins -- > Nodes  -- > New node -- > nodename: abc -- > permanent agent -- > save 

CONFIGURATION OF SALVE:

Number of executors : 3 #Number of Parallel builds
Remote root directory : /tmp #The place where our output is stored on slave sever.
Labels : one #place the op in a particular slave
useage: last option
Launch method : last option 
Host: (your privte ip)
Credentials -- > add -- >jenkins -- > Kind : ssh username with privatekey -- > username: ubuntu
privatekey : pemfile of server -- > save -- > 
Host Key Verification Strategy: last option

STEP-3: BUILD JOB ON SLAVE 

DASHBOARD -- > JOB -- > CONFIGURE -- > RESTRTICT WHERE THIS JOB RUN -- > LABEL: SLAVE1 -- > SAVE


TOMCAT:

WEBSITE: FRONTEND -- > DB IS OPT
WEBAPP: FRONTEND + BACKEND -- > DB IS MANDATORY

WEB APPLICATION SERVER/APPLICATION SERVER/APP SERVER = TOMCAT

ITS A WEB APPLICATION SERVER USED TO DEPLOY JAVA APPLICATIONS.
IT IS WRITTEN ON JAVA LANGUAGE.
AGENT: JAVA
PORT: 8080
WE CAN DEPLOY OUR ARTIFACTS.
ITS FREE AND OPENSOURCE.
Its Platform Independent.
YEAR: 1999 

war : tomcat/webapps
jar : tomcat/lib

ALTERNATIVES: NGINX, IIS, WEBSPHERE, JBOSS, GLASSFISH, WEBLOGIC

NOTE: USE SLAVE AS TOMCAT FOR TEMP

SETUP: CREATE A NEW SERVER
sudo -i
apt update
INSTALL JAVA: sudo apt install -y openjdk-17-jdk


STEP-1: DOWNLOAD TOMCAT (dlcdn.apache.org)
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.118/bin/apache-tomcat-9.0.118.tar.gz

STEP-2: EXTRACT THE FILES
tar -zxvf apache-tomcat-9.0.118.tar.gz

STEP-3: CONFIGURE USER, PASSWORD & ROLES
vim apache-tomcat-9.0.118/conf/tomcat-users.xml

 56   <role rolename="manager-gui"/>
 57   <role rolename="manager-script"/>
 58   <user username="tomcat" password="raham123" roles="manager-gui, manager-script"/>

STEP-4: DELETE LINE 21 AND 22
vim apache-tomcat-9.0.115/webapps/manager/META-INF/context.xml

STEP-5: STARTING TOMCAT
sh apache-tomcat-9.0.115/bin/startup.sh

CONNECTION:
COPY PUBLIC IP:8080 
manager apps -- > username: tomcat & password: raham123

SCRIPT:


sudo -i
apt update
INSTALL JAVA: sudo apt install -y openjdk-17-jdk



#! /bin/bash
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.115/bin/apache-tomcat-9.0.115.tar.gz
tar -zxvf apache-tomcat-9.0.115.tar.gz
sed -i '56  a\<role rolename="manager-gui"/>' apache-tomcat-9.0.115/conf/tomcat-users.xml
sed -i '57  a\<role rolename="manager-script"/>' apache-tomcat-9.0.115/conf/tomcat-users.xml
sed -i '58  a\<user username="tomcat" password="raham123" roles="manager-gui, manager-script"/>' apache-tomcat-9.0.115/conf/tomcat-users.xml
sed -i '59  a\</tomcat-users>' apache-tomcat-9.0.115/conf/tomcat-users.xml
sed -i '56d' apache-tomcat-9.0.115/conf/tomcat-users.xml
sed -i '21d' apache-tomcat-9.0.115/webapps/manager/META-INF/context.xml
sed -i '22d'  apache-tomcat-9.0.115/webapps/manager/META-INF/context.xml
sudo sh apache-tomcat-9.0.115/bin/startup.sh

DEPLOYING THE APP:
WORKSPACE IN JENKINS DASHBOARD
TARGET 
CLICK ON 
NETFLIX-1.2.2.war	

GO TO TOMCAT
CLICK ON WAR file to deploy - > CHOOSE FILE -- > UPLOAD NETFLIX WAR -- > DEPLOY 

==============================================================================================================

PIPELINE: STEP BY EXECUTION OF A PROCESS
SERIES OF EVENTS INTERLINKED WITH EACH OTHER.

code -- > build -- > test -- > artifact -- > deployment
              CI-PIPELINE                  CD-PIPELINE
 --------------------------------------  ---------------- 


IF PIPELINE FAILS ON STAGE-1 IT WONT GO FOR STAGE-2.

plugin download:
Dashboard
Manage Jenkins
Plugins
available 
pipeline stage view
install
Restart 


why to use ?
to automate the work.
to have clarity about the stage.

TYPES:
1. DECLARATIVE
2. SCRIPTED

pipeline syntax:
We use Groovy Script for jenkins Pipeline.
it consists of blocks that include stages.


SHORTCUT: PASSS

P	: PIPELINE
A	: AGENT
S	: STAGES
S	: STAGE
S	: STEPS 


SINGLE STAGE: this pipeline will have only one stage.

EX-1:
pipeline {
    agent any 
    
    stages {
        stage('abc') {
            steps {
               sh 'touch file1'
            }
        }
    }
}

EX-2:
pipeline {
    agent any 
    
    stages {
        stage('raham') {
            steps {
                sh 'touch file2'
            }
        }
    }
}

MULTI STAGE: this pipeline will have more than one stage.

pipeline {
    agent any
    
    stages {
        stage ('two') {
            steps {
                sh 'lsblk'
            }
        }
        stage ('three') {
            steps {
                sh 'lscpu'
            }
        }
        stage ('four') {
            steps {
                sh 'lsmem'
            }
        }
    }
}


CI PIPELINE:

CODE + BUILD + TEST + ARTIFACT

pipeline {
    agent any
    
     
    tools {
        maven 'maven'
    }

    stages {
        stage ('checkout') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
            }
        }
        stage ('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage ('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('artifact') {
            steps {
                sh 'mvn package'
            }
        }
    }
}


PIPELINE AS A CODE: Running more than one command/action inside a single stage.
to reduce the length of the code.
to save the time.

pipeline {
    agent any
    
     
    tools {
        maven 'maven'
    }

    stages {
        stage ('checkout') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
                sh 'mvn compile'
                sh 'mvn test'
                sh 'mvn package'
            }
        }
    }
}

MULTI STAGE PIPELINE AS A CODE: Running more than one command/action in multiple stages.


pipeline {
    agent any
    
     
    tool {
        maven 'maven'
    }

    stages {
        stage ('one') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
                sh 'mvn compile'
            }
        }
        stage ('two') {
            steps {
                sh 'mvn test'
                sh 'mvn package'
            }
        }
    }
}

PAAC OVER SINGLE SHELL: Running all the shell commands on a single shell.

pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    stages {
        stage ('checkout') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
                sh '''
                mvn compile
                mvn test
                mvn clean package
                '''
            }
        }
    }
}


INPUT PARAMETERS: BASED ON USER INPUT THE PIPELINE IS GOING TO EXECUTE.
used to avoid the mistakes.
pipeline will wait until we provide the input.

pipeline {
    agent any
    
    tools {
        maven 'maven'
    }

    stages {
        stage ('checkout') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
            }
        }
        stage ('build') {
            steps {
                sh 'mvn compile'
            }
        }
        stage ('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('artifact') {
            steps {
                sh 'mvn package'
            }
        }
        stage ('deploy') {
            input {
                message "is your inputs correct ?"
                ok "yes"
            }
            steps {
                echo "my code is deployed"
            }
        }
    }
}

NOTE: In real time providing manual approval is best practise for Jenkins pipelines.

NOTE: if we have syntax issue none of the stages will execute.


TERRAFORM  PIPELINE:

CREDENTIALS ADDING

MANAGE JENKINS
CREDENTIALS
GLOBAL
ADD CREDENTIALS
KIND: SECERTE TEXT
AWS_ACCESS_KEY_ID: VALUE -- > id: accesskey
AWS_SECRET_ACCESS_KEY: VALUE -- > id: secretkey


pipeline {
    agent any
     
    environment {
        AWS_ACCESS_KEY_ID = credentials('accesskey')
        AWS_SECRET_ACCESS_KEY = credentials('secretkey')
    }
    
    stages {
        stage('Code') {
            steps {
                git branch: 'main', url: 'https://github.com/RAHAMSHAIK007/rahamterrarepo.git'
            }
        }
        stage('init') {
            steps {
                sh 'terraform init'
            }
        }
        stage('validate') {
            steps {
                sh 'terraform validate'
            }
        }
        stage('plan') {
            steps {
                sh 'terraform plan'
            }
        }
        stage('apply') {
            input {
                message "is everything fine ?"
                ok "yes"
            }
            steps {
                sh 'terraform apply -auto-approve'
            }
        }

    }
}


NOTE: INSTALL TERRAFORM ON EC2 OR JENKINS CONSOLE

DIFF BLW SCRIPTED VS DECLARATVE

SCRIPTED: 	DECLARATIVE: 
SHORT   	LONG
NO STAGES       IT HAS STAGES
START NODE      START WITH PIPELINE
