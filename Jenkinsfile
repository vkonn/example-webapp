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
