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
                git branch: 'main', url: 'https://github.com/Kondavenkat035/farmhouse-website.git'
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
    environment {
        SCANNER_HOME = tool 'sonarqube'
    }
    steps {
        withSonarQubeEnv('sonarqube') {
            sh '''
            ${SCANNER_HOME}/bin/sonarquber \
            -Dsonar.projectKey=farmhouse-app \
            -Dsonar.projectName=farmhouse-app \
            -Dsonar.sources=. \
            -Dsonar.language=js
            '''
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
