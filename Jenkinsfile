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
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'sujan-git', url: 'https://github.com/sujanvijay/Hotstar.git']])
            }
        }
        stage('BUILD') {
        steps {
            sh 'mvn clean install -DskipTests'
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
      stage('Deploy Monitoring Stack') {
    steps {
        withKubeConfig(credentialsId: 'kubeconfig') {
            sh '''
            kubectl apply -f prometheus.yaml
            kubectl apply -f grafana.yaml
            # Deploy Node Exporter 
            kubectl apply -f node-exporter.yaml 
            # Restart Prometheus to pick up config changes 
            kubectl rollout restart deployment/prometheus
            '''
        }
      }
    }  
  }
    post {
        success {
            emailext(
                subject: "Jenkins Job '${env.JOB_NAME}' Success",
                body: "Good news! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded.\n\nCheck console output at ${env.BUILD_URL}",
                to: "sujanvijay2311@gmail.com"
            )
        }

        failure {
            emailext(
                subject: "Jenkins Job '${env.JOB_NAME}' Failed",
                body: "Alert! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) failed.\n\nCheck console output at ${env.BUILD_URL}",
                to: "sujanvijay2311@gmail.com"
            )
        }
    }
}
