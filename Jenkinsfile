pipeline {
  agent any

  environment {
    IMAGE_NAME = "yourdockerhubusername/your-image-name"
    AWS_ECR_URI = "your-aws-account-id.dkr.ecr.region.amazonaws.com/your-repo-name"
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/yourusername/your-repo.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE_NAME}")
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          script {
            sh """
              echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
              docker push ${IMAGE_NAME}
            """
          }
        }
      }
    }

    stage('Push to AWS ECR') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'aws-ecr', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_ACCESS_KEY')]) {
          script {
            sh """
              aws configure set aws_access_key_id $AWS_ACCESS_KEY
              aws configure set aws_secret_access_key $AWS_SECRET
              aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin ${AWS_ECR_URI}
              docker tag ${IMAGE_NAME} ${AWS_ECR_URI}
              docker push ${AWS_ECR_URI}
            """
          }
        }
      }
    }
  }
}
