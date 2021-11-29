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
        // stage('Build') {
        //     steps {
        //         sh "mvn clean install"
        //     }
        // }

        stage('Update Version and Tag') {
            steps {
                script {
                    // read current version 
                    pom = readMavenPom file: 'pom.xml'
                    currentVersion = pom.version

                    // read previous version
                    def filename = 'version.yaml'
                    def data = readYaml file: filename
                    previousVersion = data.version
                    
                    // read old tag
                    def filetag = 'tag.yaml'
                    def tagdata = readYaml file: filetag
                    previousTag = tagdata.tag

                    // compare current and previous version
                    def currentTag = previousTag
                    if (previousVersion == currentVersion) {
                        final oldTagLastDigit = previousTag.substring(previousTag.length()-1) as int
                        // echo "$oldTagLastDigit"
                        final deleted = previousTag.substring(0, previousTag.length()-1)
                        // echo "$deleted"
                        currentTag = deleted + (++oldTagLastDigit)
                        // echo "$currentTag"
                    }
                    else {
                        final oldTagFirstDigit = previousTag.substring(0, 1) as int
                        // echo "$oldTagFirstDigit"
                        currentTag = "" + (++oldTagFirstDigit) + ".0.1"
                        // echo "$currentTag"
                    }
                    
                    // Change old version and tag
                    data.version = currentVersion
                    sh "rm $filename"
                    writeYaml file: filename, data: data

                    tagdata.tag = currentTag
                    sh "rm $filetag"
                    writeYaml file: filetag, data: tagdata

                    // set env variable to use in next script tag
                    env.TAG = currentVersion
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // dockerImage = docker.build("meakkurt/gjg-restapi-demo")
                    docker.withRegistry(
                        'https://461902953491.dkr.ecr.eu-west-1.amazonaws.com',
                        'ecr:eu-west-1:aws-ecr') {
                            def image = docker.build('gjg-restapi-demo')
                            image.push(env.TAG)
                    }   
                }
            }
        }
        // stage('Push Docker Image') {
        //     steps {
        //         script {
        //             docker.withRegistry('', 'dockerhub') {
        //                 dockerImage.push("$env.BUILD_TAG");
        //                 dockerImage.push('latest');
        //             }
        //         }
        //     }
        // }
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
