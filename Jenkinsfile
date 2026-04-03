pipeline {
    agent any

    triggers {
        githubPush()
    }
    
    environment {
        SONARQUBE_ENV = 'sq'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'sujangit', url: 'https://github.com/sujanvijay/Hotstar.git']])
            }
        }
        stage('BUILD') {
        steps {
            sh 'mvn clean package -DskipTests'
        }
      }
      stage('JENKINS TO NEXUS') {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }
      stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }  
      stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                 }
            }
        }
      stage('Build Docker Image') {
            steps {
                sh 'docker build -t sujanvijay/myapp:latest .'
            }
        }  
      stage('Push to Docker Hub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'Docker_CRED',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
            docker push sujanvijay/myapp:latest
            '''
        }
       }
      }
      stage('Deploy to Kubernetes') {
    steps {
        withKubeConfig(credentialsId: 'kubeconfig') {
            sh 'kubectl apply -f deployment.yaml'
            sh 'kubectl apply -f service.yaml'
            }
        }
    }
  }
}
