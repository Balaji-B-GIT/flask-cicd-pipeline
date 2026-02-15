pipeline {

agent any

environment {
    DOCKER_IMAGE = "balaji782/flask-cicd-pipeline:latest"
    SONAR_AUTH_TOKEN = credentials('sonar-token')
}

stages {

    stage('SonarQube Analysis') {
    agent {
        docker {
            image 'sonarsource/sonar-scanner-cli'
            args '--network=host -u root'
        }
    }
    steps {
        withSonarQubeEnv('sonarqube') {
            sh '''
            sonar-scanner \
              -Dsonar.projectKey=flask-cicd-pipeline \
              -Dsonar.sources=. \
              -Dsonar.userHome=.sonar \
              -Dsonar.host.url=http://localhost:9000 \
              -Dsonar.login=$SONAR_AUTH_TOKEN
            '''
        }
    }
}

    stage('Build Docker Image') {
        steps {
            sh 'docker build -t $DOCKER_IMAGE .'
        }
    }

    stage('Push Docker Image') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub',
                usernameVariable: 'USERNAME',
                passwordVariable: 'PASSWORD'
            )]) {
                sh '''
                echo $PASSWORD | docker login -u $USERNAME --password-stdin
                docker push $DOCKER_IMAGE
                '''
            }
        }
    }

    stage('Deploy to Kubernetes') {
        steps {
            sh 'kubectl apply -f k8s/deployment.yaml'
        }
    }
}
}
