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
        git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/Joseph-peemi/Board-game.git'
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
          waitForQualityGate abortPipeline: false
          credentialsId: 'sonar-token'
        }
      }
        }
        stage('Build') {
      steps {
        sh 'mvn package'
      }
       }
  }
}
