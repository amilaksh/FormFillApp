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
                withAWS(region: "${AWS_REGION}", credentials: 'aws-creds') {
                    sh '''
                        aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                        kubectl apply -f deployment.yml
                    '''
                }
            }
        }
    }  // <-- yeh "stages" block close karta hai

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

