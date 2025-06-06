pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        scannerHome = tool 'sonar-scanner'
        IMAGE = "codersdiary/nodejs"
        VERSION = "${new Date().format('yyyyMMddHHmmss')}"
        CONTAINER_NAME = "java-container"
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Shubham-6364/Jenkins-SonarQube-trivy.git'
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
                sh "trivy fs --format table -o trivy-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=javaproject \
                        -Dsonar.projectName=javaproject \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test/java \
                        -Dsonar.java.binaries=target"
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'SonarQubeServer'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE}:${VERSION} ."
                sh "docker tag ${IMAGE}:${VERSION} ${IMAGE}:latest"
            }
        }

        stage('Docker image scan') {
            steps {
                sh "trivy image --format table -o trivy-report.html ${IMAGE}:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        sh "docker push ${IMAGE}:${VERSION}"
                        sh "docker push ${IMAGE}:latest"
                    }
                }
            }
        }

        stage('Redeploy Container') {
            steps {
                script {
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    sh "docker run -d --name ${CONTAINER_NAME} -p 8081:8080 ${IMAGE}:${VERSION}"
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/trivy-*.html', allowEmptyArchive: true
        }
    }
}
