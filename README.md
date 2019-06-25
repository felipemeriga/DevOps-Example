# DevOps-Example
This is a sample Spring Boot Application, used to explain the Jenkins pipeline, in creating a full CI/CD flow using docker too.

# Jenkins 
Jenkins is an open source automation server written in Java. Jenkins helps to automate the non-human part of the software development process,
 with continuous integration and facilitating technical aspects of continuous delivery. It is a server-based system that runs in servlet containers 
 such as Apache Tomcat.
 
# Docker 

Docker is a collection of interoperating software-as-a-service and platform-as-a-service offerings that employ operating-system-level virtualization 
to cultivate development and delivery of software inside standardized software packages called containers. The software that hosts the containers 
is called Docker Engine.

# Using Jenkins with Docker
First of all to use jenkins with docker, we will have to know that as we are running Jenkins inside a docker container, and we need access to docker to
build our services images, so first of all let's build a Jenkins image with docker installed inside it. Let's check the dockerfile to 
build this Jenkins image with docker inside, you will just have to put this file into any folder, and run the docker build command.

# Dockerfile for Jenkins
```
from jenkins/jenkins:lts
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get install docker-ce=17.12.1~ce-0~debian -y
RUN usermod -aG docker jenkins
```

Just place this Dockerfile in any folder and run the following commands:

$ docker image build -t jenkins-docker .

Now that the docker image has already been built, we can run the Jenkins in a docker container with the command:

$ docker container run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock jenkins-docker

# Building Spring Boot Application
The Sample application built here has the maven jar plugin, so in order to build that as a jar, we just have to do the command:

$ mvn clean install

# Using with AWS ECS and ECR

First of all you will have to push your image to Amazon ECR, so in order to do this via command line interface, we have to have
the aws cli installed on our computer. After that we will have to synchronize the docker login with the AWS account, with the command:

$(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)

After that you will sync the two accounts, and are able to build and push your image. So, just build the image:

$ docker build -t devops-service . 

Tag you image:

$ docker tag devops-service:latest $AWS_IAM_USER_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/devops-service:latest

Finally run thios command for pushing the image:

docker push $AWS_IAM_USER_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/devops-service:latest
