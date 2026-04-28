pipeline {
    agent any

    tools {
        jdk 'jdk21'
        nodejs 'node23'
 }
        environment {
        SONARQUBE_ENV = 'sonarqube'
        }          
  stages {
      stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/CVN9696/zomato-nodejs.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Package Artifact') {
            steps {
                sh 'zip -r farmhouse-build.zip build/'
            }
        }
      stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome = tool 'SonarScanner'

            withSonarQubeEnv('sonarqube') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=farmhouse \
                -Dsonar.projectName=farmhouse-App \
                -Dsonar.sources=src \
                -Dsonar.projectVersion=${BUILD_NUMBER}
                """
            }
        }
    }
}

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
  }
}
