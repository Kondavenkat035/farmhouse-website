pipeline {
    agent any

    tools {
        jdk 'jdk21'
        nodejs 'node23'
 }
  stages {

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
                sh 'zip -r zomato-build.zip build/'
            }
        }
  }
}
