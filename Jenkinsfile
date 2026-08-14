pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"
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

        stage('Docker Build') {
            steps {
                sh '''
                    which docker
                    docker --version
                    docker build -t formfillapp:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    docker stop formfillapp-container || true
                    docker rm formfillapp-container || true
                    docker run -d \
                      --name formfillapp-container \
                      -p 8081:8080 \
                      formfillapp:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
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

