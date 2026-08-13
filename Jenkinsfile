// Ye poora pipeline ka block hai, Jenkins isi ko padhta hai
pipeline {

    // agent any ka matlab - Jenkins ka jo bhi system available ho usi pe chalao
    // Tumhare case me tumhara MacBook hi agent hai
    agent any

    // options = pipeline kaise chalega uski setting
    options { 
        // Jenkins by default khud hi code checkout karta hai, aur tumne neeche 
        // Checkout stage me bhi likha hai. Isse 2 baar checkout ho raha tha.
        // skipDefaultCheckout(true) se double checkout band ho jayega, time bachega
        skipDefaultCheckout(true) 
    }
    
    // environment = saare stages ke liye global variables
    environment {
        // PATH = Jenkins ko batao ki commands kahan dhoondhne hai
        // Tumhara docker /usr/local/bin me tha par Jenkins wahan dekh nahi raha tha
        // Isiliye humne uska rasta yahan add kar diya, ab docker mil jayega
        PATH = "/usr/local/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    // stages = tumhara kaam step-by-step yahan hota hai
    stages {
        
        // STAGE 1: GitHub se code lana
        stage('Checkout') { 
            steps { 
                // checkout scm = Source Code Management
                // Matlab https://github.com/amilaksh/FormFillApp.git se latest code pull karo
                checkout scm 
            } 
        }

        // STAGE 2: Code ko test karna
        stage('Maven Test') { 
            steps { 
                // sh = shell command chalao
                // /opt/homebrew/bin/mvn clean test = 
                // clean = purane build files delete karo
                // test = server ke andar jo TestGreeter.java hai usko chalao
                // Tumhare log me 2 tests pass hue the - yahi wala stage hai
                sh '/opt/homebrew/bin/mvn clean test' 
            } 
        }

        // STAGE 3: Code ka package banana (jar/war file)
        stage('Maven Package') { 
            steps { 
                // package = code ko jar aur war file me convert karo
                // -DskipTests = test dubara mat chalao, upar wale stage me ho gaya hai
                // Isse server/target/server.jar aur webapp/target/webapp.war banta hai
                sh '/opt/homebrew/bin/mvn package -DskipTests' 
            } 
        }

        // STAGE 4: Test ki report dikhana
        stage('JUnit Report') { 
            steps { 
                // junit = Jenkins ko test ka result dikhao
                // server/target/surefire-reports/*.xml = Maven test report ka rasta
                // Isse Jenkins me blue/green graph aata hai kitne test pass hue
                junit 'server/target/surefire-reports/*.xml' 
            } 
        }
        
        // STAGE 5: Docker image banana
        stage('Docker Build') {
            steps {
                sh '''
                    // which docker = check karo docker kahan installed hai, debug ke liye
                    which docker
                    
                    // docker --version = kaunsa version hai wo print karo
                    docker --version
                    
                    // docker build = Dockerfile se image banao
                    // -t formfillapp:${BUILD_NUMBER} = image ka naam do
                    // formfillapp = image ka naam, ${BUILD_NUMBER} = Jenkins ka build number jaise 5, 6, 7
                    // . = current folder me Dockerfile dhoondho
                    docker build -t formfillapp:${BUILD_NUMBER} .
                '''
            }
        }

        // STAGE 6: Docker container chalana
        stage('Docker Deploy') {
            steps {
                sh '''
                    // Agar pehle se same naam ka container chal raha hai toh usko roko
                    // || true = agar container nahi mila toh error mat do, aage badho
                    docker stop formfillapp-container || true
                    
                    // Purane container ko delete karo taaki naya bana sake
                    docker rm formfillapp-container || true

                    // Naya container chalao
                    // -d = background me chalao (detached)
                    // --name formfillapp-container = container ko naam do
                    // -p 8081:8080 = tumhare Mac ka 8081 port, container ke 8080 se jodo
                    // Matlab localhost:8081 pe app khulega
                    // formfillapp:${BUILD_NUMBER} = kaunsi image chalani hai
                    docker run -d --name formfillapp-container -p 8081:8080 formfillapp:${BUILD_NUMBER}
                '''
            }
        }
    }
}
