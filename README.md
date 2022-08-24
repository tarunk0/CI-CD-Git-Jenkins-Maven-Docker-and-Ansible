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
   - Connect to instances and set password for user "ansadmin".
```
   passwd ansadmin
```
   - Give sudo permission to user "ansadmin"
```
   vi sudo
   ansadmin ALL=(ALL)  NOPASSWD:ALL
```
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
   
