# CI-CD-Git-Jenkins-Maven-Docker-and-Ansible


![Screenshot 2022-08-24 at 4 28 37 AM](https://user-images.githubusercontent.com/92631457/186288700-8964ac49-9d53-4e4d-a184-0eb12d45ce24.png)

![Screenshot 2022-08-24 at 4 29 21 AM](https://user-images.githubusercontent.com/92631457/186288774-a05b7e7b-a13f-4b72-bdd6-3a0b82293f55.png)

## STEP: 1 
   ### Creating User on Jenkins, Docker and Ansible Instances:
   - Create 3 Instances, for this tutorial I have created all of them in GCP.
   - Jenkins Instance Name: jenkins
   - Ansible Instance Name: ansible-control-node
   - Docker Instance Name: docker-host
   - Generate SSH Keys and Create User "ansadmin"
      - Generate keys using putty key generator or similar tool. 
      - Key comments: ansadmin
      - Save private and public key. 
      - Copy Public Key to VM(Edit mode) in SSH Key block
      
      ![image](https://user-images.githubusercontent.com/92631457/186561759-4ff58ab9-0e62-4ae6-8e7c-1e6b8aef983b.png)

      - Copy Public Key to VM(Edit mode) in SSH Key block  
   - Connect to instances and set password for user "ansadmin".
```
   passwd ansadmin
```
   - Give sudo permission to user "ansadmin"
```
   vi sudo
   ansadmin ALL=(ALL)  NOPASSWD:ALL
```
   ![image](https://user-images.githubusercontent.com/92631457/186561531-43672b9d-9e0e-4d6c-8e9c-fa338146b363.png)
   
## STEP: 2
   ### Java Installation on all three instances, as It is a java based project:
```
  apt update
  apt install openjdk-8-jdk -y
  # Or you can run 
  apt install openjdk-8-jre-headless
  # check java version
  java -version
```
   ### Enable the Password Authentication on all three instances:(By default it is set to No)
   
``` 
   vi /etc/ssh/sshd_config
   PasswordAuthentication yes #by default it is set to no
   service sshd reload #to refresh the changes
```
![Screenshot 2022-08-25 at 8 13 56 AM](https://user-images.githubusercontent.com/92631457/186562504-92021aec-c3cb-481c-8635-c269c3251139.png)

   
 ## STEP: 3
  ### Maven Installation and configuration on Jenkins Instance:
     
```
   apt update
   cd /opt
   wget https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz 
   #replace with latest version from  this link: https://maven.apache.org/download.cgi

   tar -xvzf apache-maven-3.8.6-bin.tar.gz
   mv apache-maven-3.8.6-bin maven

   #set maven path of gcp ubuntu server for jenkins
   vi /etc/profile
   M2_HOME=/opt/maven
   M2=$M2_HOME/bin
   PATH=$PATH:$JAVA_HOME:$M2_HOME:$M2:$HOME/bin
   export PATH
```
![image](https://user-images.githubusercontent.com/92631457/186562736-3e9e885c-0ba2-43b7-b75c-41117a47e947.png)

```
   #for refreshing
   source /etc/profile
   echo $PATH
   echo $M2
  
```
   ### Jenkins Installation and configuration on Jenkins Instance:

```
  # This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system:
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key |      sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    
    #Then add a Jenkins apt repository entry:
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    
    #Update your local package index, then finally install Jenkins:
    sudo apt-get update
    sudo apt-get install fontconfig openjdk-11-jre
    sudo apt-get install jenkins
    
    # Give sudo permission to the jenkins user
    vi /etc/sudoers
   
    #add the following line beside root
    jenkins ALL=(ALL) NOPASSWD: ALL
```
![image](https://user-images.githubusercontent.com/92631457/186561374-c478fb09-e8d9-4b7c-9f65-c41c8e491c8a.png)
      
```
    #restart jenkins
    systemctl restart jenkins
   
    #Test Jenkins
    http://<jenkins-public-ip.:8080
```
 
   ### Jenkins Plugins installation and configuration of Jenkins:
  
   - Plugins to be installed. Default Plugins installation is not required as we need some specific plugins.
     - Github
     - Maven Invoker
     - Maven Integration
     - Publish over SSH. 
 
 
  ## STEP: 4
   ### Ansible Installation and configuration on Ansible Named Instance:   
   
``` 
   #Ansible installation
   apt install software-properties-common
   apt-add-repository --yes --update ppa:ansible/ansible
   apt install ansible -y
   ansible --version
```
   
```
   #add the targets groups in the ansible hosts file
   vi /etc/ansible/hosts
   [all_hosts]
   localhost
   <private-ip-docker> #this is for the configuration to be done on docker as ansible is a configuration management tool
```
![image](https://user-images.githubusercontent.com/92631457/186563280-e43910f1-015d-4221-9b76-1da5aff09f9d.png)

```
   #create a directory in /opt in Ansible server using user 'ansadmin'
   sudo su - ansadmin
   cd /opt
   sudo mkdir docker
   sudo chown -R ansadmin:ansadmin /opt/docker
   ls -l /opt
```
   
## STEP: 5
   ### Docker Installation and configuration on Docker host  and Ansible Named Instance:
   
 - Install docker and start docker service
```
   apt install docker.io -y
   docker --version
   service docker start
   service docker status
```
 - Add a user to docker group to manage docker
```
   usermod -aG docker
   ansadmin
```

## STEP: 6
   ### Create a Password less connection between Ansible and Docker hosts Instances:
   
   - Log into Ansible Instance using user ansadmin and generate ssh key:
```
    sudo su - ansadmin
    ssh-keygen
```
   - Copy Keys onto All Ansible managed Hosts - Docker Instance:
``` 
    ssh-copy-id ansadmin@<private-ip-docker-server>
    ssh ansadmin@<private-ip-docker-server>
    exit
```
   - Copy Keys to Local Ansible Instance
``` 
    ssh-copy-id localhost
```
   - Validation Test
``` 
   ansible all -m ping
``` 
![image](https://user-images.githubusercontent.com/92631457/186560898-3ddd7001-f9a0-4902-b91c-19a5aa699cab.png)

   
 ## STEP: 7
   ### Setup connection between Jenkins and Ansible and also set the MAVEN path
   
   - Manage Jenkins - Configuration System - Publish over SSH
     - ADD
     - Name: ansible-server (Name of ansible system)
     - Username: ansadmin
     - Advances: select password authentication
     - Password: ansadmin
     - Test connection
     
     ![image](https://user-images.githubusercontent.com/92631457/186560714-69ad9cbf-e7a4-430f-ad6f-0552b31ee878.png)


   - Mangage Jenkins - Global Tool configuration - Maven
     - Name: maven
     - MAVEN_HOME: /opt/maven/
     
     ![image](https://user-images.githubusercontent.com/92631457/186560358-477f03a2-554c-4a98-86cf-a641e115b264.png)


   ### Github Repository url:
    
   - Repository Url:
     - Your github url --
     - Branch: /main
     
     ![image](https://user-images.githubusercontent.com/92631457/186559828-49eb9f29-2558-40ee-8894-c527aa25c4fd.png)

   
   - Change hosts file in github and add docker instance private IP:
   
   ![image](https://user-images.githubusercontent.com/92631457/186559887-ed622090-dc85-49b4-ae0c-919d9b94510c.png)

   - Change hosts file in github and add docker instance private IP:
   - Login with Docker-Hub Credentials on ansible istance using "root user"
     - docker login
     - Username: tarunk0
     - Password: *********
     
     ![image](https://user-images.githubusercontent.com/92631457/186560259-ba5a0e11-63f4-4436-a401-413b726e723a.png)

    
## STEP: 8
   ### Jenkins(Maven Project) Job Configuration:
   
   - Project name: deploy_on_container_using_ansible
   - Build - Root POM - pom.xml
     - Goals and Options: clean install package
     
     ![image](https://user-images.githubusercontent.com/92631457/186559760-281d12d0-e431-45f5-b22a-ca7166412a1b.png)

   - Post build actions:
     - Add Post build Action - Send build artifacts over SSH
     - SSH Server
       - Name: ansible-server
       - Source file: Dockerfile, hosts, create-simple-devops-project.yml, create-simple-devops-image.yml (Jenkins Instance path - /var/lib/jenkins/workspace/Deploy_on_container_using_ansible)
       - Remote Directory: //opt/docker
       
       ![image](https://user-images.githubusercontent.com/92631457/186559709-856b6bb1-33c1-4cfc-8e17-9f2ab70389e4.png)

     - SSH Server
       - Name: ansible-server
       - Source file: webapp/target/*.war
       - Remove prefix: webapp/target
       - Remove Directory: //opt//docker
       - Exec Command -
```
    ansible-playbook -i /opt/docker/hosts /opt/docker/create-simple-devops-image.yml --limit localhost;
    ansible-playbook - /opt/docker/hosts  /opt/docker/create-simple-devops-project.yml --limit <docker-private-ip>;
```
![image](https://user-images.githubusercontent.com/92631457/186559622-df9cf972-0536-423e-9634-0b497f8b856c.png)

   - Test the URL

```
   http://<docker-public-ip>:8080/webapp
```
   - Dockerfile
```
   #pull tomcat base image from dockerhub
   FROM tomcat:latest
    
   #maintainer
   MAINTAINER "Tarun K."
   
   #copy war files on container
   COPY /webapp.war /usr/local/tomcat/webapps
```
   - Ansible Playbook:
``` 
   vim create-simple-devops-image.yml
```
     
``` 
   ---
   
   - hosts:
     become: yes
     tasks:
       - name: create docker image using a war file
         command: docker build -t simple-devops-image: latest .
       args:
         chdir: /opt/docker
       - name: create tag to image
         command: docker tag simple-devops-image tarunk0/simple-devops-image
       - name: push image to docker hub
         command: docker push tarunk0/simple-devops-image
       - name: remover docker images from ansible server
         command: docker rmi tomcat simple-devops-image:latest tarunk0/simple-devops-image
         ignore_errors: yes
 ```
   - Ansible Playbook:
``` 
   vim create-simple-devops-project.yml
```
 
```
   ---
- hosts: all
  become: yes

  tasks:

    - name: stop current running container
      command: docker stop simple-devops-container
      ignore_errors: yes

    - name: remove stopped docker container
      command: docker rm simple-devops-container
      ignore_errors: yes

    - name: remove current docker image
      command: docker rmi ameintu/simple-devops-image:latest
      ignore_errors: yes

    - name: pull docker image from dockerhub
      command: docker pull ameintu/simple-devops-image:latest

    - name: creating docker container using simple-devops-image
      command: docker run -d --name simple-devops-container -p 8080:8080 ameintu/simple-devops-image:latest
 ```
    
## STEP: 9
   ### Webhook configuration to automat the complete flow of CI/CD:
   
   - Adding webhook to github repository
     - Payload url: http://<jenkins-public-ip>:8080/github-webhook/
     ![image](https://user-images.githubusercontent.com/92631457/186559441-e6039c81-f05a-457d-8aa4-fbbc1e752f8d.png)

     
     - Enable 'Github hook trigger for GIT SCM polling in Jenkins project
     ![image](https://user-images.githubusercontent.com/92631457/186559277-1f94d3af-98b8-4365-b6c1-ca9ecae0c720.png)

   
   - Testing
     - Clone project in local
     - Make the changes and commit them  
     - Build will automatically trigger. 
     - Changes will be reflected in the webapp.
   
   - Test the URL:
     - http://<public-ip-docker>:8080/webapp
  
  
  
  
![image](https://user-images.githubusercontent.com/92631457/186559179-a0b1d700-38ef-4014-9a77-c55b9df5fc5c.png)

![image](https://user-images.githubusercontent.com/92631457/186559138-7a3d2e3f-fd04-4dcf-b48d-ce04ffca38c3.png)
 
  
  
  
  
  
  
  
  
  
  
     
     
     
     
     
     
     
     
     
     
     
     
     
   
   
