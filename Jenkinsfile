pipeline {
  agent any

  environment {
    IMAGE_NAME = "aravinthexe/mlflow_app_v1"
    AWS_ECR_URI = "994390684427.dkr.ecr.eu-north-1.amazonaws.com/mlflow"
    REGION = "eu-north-1"
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
          docker.build("${IMAGE_NAME}:latest")
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub',
          passwordVariable: 'DOCKER_PASSWORD',
          usernameVariable: 'DOCKER_USERNAME'
        )]) {
          script {
            bat """
              echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
              docker push ${IMAGE_NAME}
            """
          }
        }
      }
    }

    stage('Push to AWS ECR') {
      steps {
         withCredentials([
          string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          script {
            // sh """
            //   aws configure set aws_access_key_id $AWS_ACCESS_KEY
            //   aws configure set aws_secret_access_key $AWS_SECRET
            //   aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin ${AWS_ECR_URI}
            //   docker tag ${IMAGE_NAME} ${AWS_ECR_URI}
            //   docker push ${AWS_ECR_URI}
            // """
            bat """
              aws configure set aws_access_key_id %AWS_ACCESS_KEY_ID%
              aws configure set aws_secret_access_key %AWS_SECRET_ACCESS_KEY%
              aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 994390684427.dkr.ecr.eu-north-1.amazonaws.com
              docker tag aravinthexe/mlflow_app_v1:latest 994390684427.dkr.ecr.eu-north-1.amazonaws.com/fastapi:latest
              docker push 994390684427.dkr.ecr.eu-north-1.amazonaws.com/fastapi:latest
            """
          }
        }
      }
    }

    stage('Deploy to ECS') {
      steps {
        withCredentials([
          string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
          string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
          script {
            bat """
              aws configure set aws_access_key_id %AWS_ACCESS_KEY_ID%
              aws configure set aws_secret_access_key %AWS_SECRET_ACCESS_KEY%
              aws ecs update-service ^
                --cluster timeseries-forecasting ^
                --service timeseries-forecasting-service-6nxf55dq ^
                --force-new-deployment ^
                --region eu-north-1
            """
          }
        }
      }
    }
  }
}
