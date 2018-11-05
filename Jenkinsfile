pipeline {
    agent any
    stages{
        stage('Build') {
            steps{
                aws ecr get-login --no-include-email --region us-east-1 | bash

				docker build --tag 415734504350.dkr.ecr.us-east-1.amazonaws.com/project-build:$(git rev-parse HEAD) -f Dockerfile.builder .

				docker push 415734504350.dkr.ecr.us-east-1.amazonaws.com/project-build:$(git rev-parse HEAD)

				docker run --rm -v "$PWD:/work" 415734504350.dkr.ecr.us-east-1.amazonaws.com/project-build:$(git rev-parse HEAD) bash -c "cd /work; lein uberjar"

				docker build -t 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:$(git rev-parse HEAD) .

				docker push 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:$(git rev-parse HEAD)
				
				#!/bin/bash
				#update the yum cache
				sudo yum update -y

				#install java
				sudo yum install java-1.8.0-openjdk -y

				#install git
				sudo yum install git -y

				sudo yum install docker -y
				sudo service docker start
				sudo chkconfig docker on
				sudo usermod -aG docker jenkins
				sudo usermod -aG docker ec2-user
				sudo yum install git -y
				sudo reboot
				
				
				
				sh 'echo "We are generating artifacts for ${BUILD_NUMBER}" > output.txt'
				#!/bin/bash
				sudo yum update -y
				sudo yum remove java
				sudo yum install java-1.8.0 -y
				sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
				sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
				sudo yum install jenkins -y
				sudo service jenkins start
				sudo chkconfig jenkins on
				sudo yum install docker -y
				sudo service start docker
				sudo chkconfig docker on
				sudo usermod -aG docker jenkins
				sudo usermod -aG docker ec2-user
				sudo yum install git -y
				sudo yum install nginx -y
				sudo service start nginx
				sudo chkconfig nginx on
				sudo reboot
            }
        }
        stage('Archive and Test') {
            steps {
                archiveArtifacts artifacts: 'output.txt', onlyIfSuccessful: true
            }
		}
        stage('Deploy') {
            steps {
                archiveArtifacts artifacts: 'output.txt', onlyIfSuccessful: true
				
				aws ecr get-login --no-include-email --region us-east-1 | bash
				docker pull 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:$TAG_TO_DEPLOY
				docker tag 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:$TAG_TO_DEPLOY 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:release
				docker push 415734504350.dkr.ecr.us-east-1.amazonaws.com/project:release
				aws ec2 reboot-instances --region us-east-1 --instance-ids $PRODUCTION_INSTANCE
            }
        }
    }
}
