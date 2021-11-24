pipeline {
    agent {
       label {
           label "Java"
       } 
    }

    environment {
        dockerHome = tool 'docker'
        mavenHome = tool 'maven'
        PATH = "$dockerHome/bin:$mavenHome/bin:$PATH"
    }

    stages {
        stage('Checkout') {
            steps {
                sh "mvn --version"
                sh "docker --version"
                echo "Build"
                echo "$PATH"
                echo "Build number - $env.BUILD_NUMBER"
                echo "Build tag - $env.BUILD_TAG"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        // stage('Test') {
        //     steps {
        //         sh "mvn test"
        //     }
        // }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("meakkurt/gjg-restapi-demo")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub') {
                        dockerImage.push("$env.BUILD_TAG");
                        dockerImage.push('latest');
                    }
                }
            }
        }
    }
    // Good when you want to clean up and etc
    // post {
    //     always {
    //         echo "Runs always"
    //     }
    //     success {
    //         echo "Runs when successful"
    //     }
    //     failure {
    //         echo "Runs when fail"
    //     }
    // }
}
