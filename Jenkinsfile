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
            steps { checkout scm } 
        }
        stage('Maven Test') { 
            steps { sh '/opt/homebrew/bin/mvn clean test' } 
        }
        stage('Maven Package') { 
            steps { sh '/opt/homebrew/bin/mvn package -DskipTests' } 
        }
        stage('JUnit Report') { 
            steps { junit 'server/target/surefire-reports/*.xml' } 
        }
        stage('Docker Build') {
            steps {
                sh '''
                    # check karo docker kahan hai
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
                    docker run -d --name formfillapp-container -p 8081:8080 formfillapp:${BUILD_NUMBER}
                '''
            }
        }
    }
}
