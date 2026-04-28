pipeline {
    agent any

    tools {
        jdk 'jdk21'
        nodejs 'node23'
 }
        environment {
        SONARQUBE_ENV = 'sonarqube'
        DOCKER_IMAGE = "kondavenkat035/farmhouse"
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
                sh 'npm install --save-dev jest'
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
            ${SCANNER_HOME}/bin/sonar-scanner \
            -Dsonar.projectKey=farmhouse-app \
            -Dsonar.projectName=farmhouse-app \
            -Dsonar.sources=. \
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
            -Dsonar.language=js
            '''
        }
    }
}

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
      stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                    curl -v -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file farmhouse-build.zip \
                    http://localhost:8081/repository/farmhouse/farmhouse-build-${BUILD_NUMBER}.zip
                    '''
                }
            }
        }
      stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .
                docker tag $DOCKER_IMAGE:${BUILD_NUMBER} $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:${BUILD_NUMBER}
                    docker push $DOCKER_IMAGE:latest
                    docker logout
                    '''
                }
            }
        }
      stage('Deploy to Kubernetes (EKS)') {
            steps {
                withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                
                        echo "Deploying to EKS..."

                        kubectl apply -f k8s/deployment.yml
                        kubectl apply -f k8s/service.yml

                        echo "Deployment status:"
                        kubectl get pods
                        kubectl get svc
                    '''
                }
            }
        }

        stage('Show Application URL') {
            steps {
                withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        echo "Fetching LoadBalancer URL..."

                        for i in {1..2}; do
                            URL=$(kubectl get svc bookstore-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' 2>/dev/null)

                            if [ "$URL" != "" ]; then
                                echo "========================================"
                                echo "Application Deployed Successfully 🚀"
                                echo "URL: http://$URL"
                                echo "========================================"
                                exit 0
                            fi

                            echo "Waiting for LoadBalancer... attempt $i/20"
                            sleep 5
                        done

                        echo "ERROR: LoadBalancer not ready. Check AWS console."
                    '''
                }
            }
        }
    }
}
