pipeline {
    agent {
       label {
           label "Java"
       } 
    }

    stages {
        stage('Build') {
            steps {
                echo "Build"
            }
        }
        stage('Test') {
            steps {
                echo "Test, but modified"
            }
        }
        stage('Integration Test') {
            steps {
                echo "Integration Test"
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
