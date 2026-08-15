pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"
        DOCKERHUB_USER = "amilaksh"
        DOCKER_IMAGE = "${DOCKERHUB_USER}/formfillapp:latest"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "formfillapp-cluster"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Test') {
            steps {
                sh '/opt/homebrew/bin/mvn clean test'
            }
        }

        stage('JUnit Report') {
            steps {
                junit testResults: 'server/target/surefire-reports/*.xml',
                      allowEmptyResults: false
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        /opt/homebrew/bin/mvn \
                          org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar \
                          -Dsonar.projectKey=FormFillApp
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Maven Package') {
            steps {
                sh '/opt/homebrew/bin/mvn package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh '''
                    docker buildx create --use || true
                    docker buildx build --platform linux/amd64,linux/arm64 \
                      -t $DOCKER_IMAGE --push .
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                    kubectl apply -f deployment.yml
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f || true'
        }
        success {
            emailext(
                subject: "SonarQube Report - FormFillApp",
                body: """SonarQube Quality Gate PASSED.
Dashboard: http://localhost:9000/dashboard?id=FormFillApp""",
                to: "amiteshranjan@outlook.com"
            )
        }
        failure {
            emailext(
                subject: "SonarQube Report - FormFillApp",
                body: "Pipeline failed. Please check Jenkins logs.",
                to: "amiteshranjan@outlook.com"
            )
        }
    }
}

