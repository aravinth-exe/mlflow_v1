pipeline {
  agent any

  environment {
    IMAGE_NAME = "aravinth.exe/fastapi_app_v1"
    AWS_ECR_URI = "994390684427.dkr.ecr.eu-north-1.amazonaws.com/mlflow"
  }

  stages {
    // stage('Checkout') {
    //   steps {
    //     git 'https://github.com/aravinth-exe/mlflow_v1.git', branch: 'main'
    //   }
    // }

    stage('Checkout') {
      steps {
        checkout scm
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
            // sh """
            //   aws configure set aws_access_key_id $AWS_ACCESS_KEY
            //   aws configure set aws_secret_access_key $AWS_SECRET
            //   aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin ${AWS_ECR_URI}
            //   docker tag ${IMAGE_NAME} ${AWS_ECR_URI}
            //   docker push ${AWS_ECR_URI}
            // """
            bat """
              aws configure set aws_access_key_id $AWS_ACCESS_KEY
              aws configure set aws_secret_access_key $AWS_SECRET
              aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin ${AWS_ECR_URI}
              docker tag ${IMAGE_NAME} ${AWS_ECR_URI}
              docker push ${AWS_ECR_URI}
            """
          }
        }
      }
    }
  }
}
