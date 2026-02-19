pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Checkout') {
      steps {
        git branch: 'main', credentialsId: 'github', url: 'https://github.com/Joseph-peemi/Board-game-App.git'
      }
        }

        stage('Compile') {
      steps {
        sh 'mvn compile'
      }
        }

        stage('Test') {
      steps {
        sh 'mvn test'
      }
        }

        stage('File System Scan') {
      steps {
        sh 'trivy fs --format table -o trivy-fs-report.html .'
      }
        }

        stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar') {
          sh '''${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Board-game \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=your-sonar-token'''
        }
      }
        }

        stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
        }
      }
        }

        stage('Build') {
      steps {
        sh 'mvn package'
      }
        }

        stage('Publish to Nexus') {
      steps {
        withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
          sh 'mvn package'
        }
      }
        }

        stage('Docker Image Scan') {
      steps {
        sh 'trivy image --format table -o trivy-docker-report.html abuchijoe/board-game:latest'
      }
        }

        stage('Build and Push Docker Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
            sh 'docker build -t abuchijoe/board-game:latest .'
            sh 'docker push abuchijoe/board-game:latest'
          }
        }
      }
        }

        stage('Deploy to Kubernetes') {
      steps {
        withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.90.111:6443') {
          sh 'kubectl apply -f deployment-service.yaml'
        }
      }
        }

        stage('Verify the Deployment') {
      steps {
        withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.90.111:6443') {
          sh 'kubectl get pods -n webapps'
          sh 'kubectl get svc -n webapps'
        }
      }
        }
    }
}
