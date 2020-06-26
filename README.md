# CI/CD Pipeline Project

A complete ci/cd pipeline for basic NodeJS app.

* Stack: Git -> Jenkins -> Sonarqube (report to postgresql) -> Docker Build/Push -> Kubernetes (deployment) 

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

What things you need to pre-install:
* Virtual Box
* Vagrant
* Docker
* Docker-compose

### Installing

A step by step guide that tell you how to get a development env running:

1. Clone git repository
```
git clone https://github.com/rijulrg/ci-cd.git
```
2. Build custom jenkins image with docker and kubectl
```
docker build -f jenkins-custom/Dockerfile -t {custom_name} .
```
3. Prepare bind mount directories for jenkins and postgresql data
```
mkdir /var/jenkins_home
chown -R 1000:1000 /var/jenkins_home
mkdir /var/postgres_data
chown -R 1000:1000 /var/postgres_data
```
4. Bring up the Jenkins, Sonarqube and Postgresql.
```
cd stack-jenkins-sonar-db/
docker-compose up -d

* Note: Run "systemctl -w vm.max_map_count = 262144" as it is required by sonarqube
        Don't forget to change jenkins container image field value with {custom_name} in docker-compose.yaml 
```
5. Download the require plugins for jenkins (other than suggested ones):

* [SonarQube Scanner](https://plugins.jenkins.io/sonar/)
* [Kubernetes CLI](https://plugins.jenkins.io/kubernetes-cli/)
* [CloudBees Docker Build and Publish](https://plugins.jenkins.io/docker-build-publish/)
* [Docker Pipeline](https://plugins.jenkins.io/docker-workflow/)
* [NodeJS](https://plugins.jenkins.io/nodejs/)

6. Spin up a local kubernetes cluster:
```
cd k8s/local_cluster/
vagrant up
vagrant ssh {master/workerNode1/workerNode2} # accordingly for ssh into respective nodes, just for verifying

* Note: verify k8s server ip is reachable from jenkins pod
```

7. Store the required credentials in jenkins credential store: (http://{ip}:8080)

* Github (Username with password): Your personal credentials
* Docker Hub (Username with password): Your personal credentials
* Kubernetes (Secret file): Respective kube.config file with Certificate Authority data
* Sonarqube  (Secret text): Login token generate on sonar server (http://{ip}:9000)

8. Create a job in jenkins:
```
New Item -> Pipeline -> Pipeline from scm ->  SCM: git -> ADD Repository URL + Credentials -> Save -> Build Now

Note: User this repo url: "https://github.com/rijulrg/ci-cd.git" for testing pupose as it contains a basic nodejs app as well.

** Pipeline File: ./Jenkinsfile
** Dockerfile for NodeJS app: ./Dockerfile
** Files related to  NodeJS app: ./package.json, ./server.js, and ./test/*
```

## Author

* **Rijul Gogia** 
