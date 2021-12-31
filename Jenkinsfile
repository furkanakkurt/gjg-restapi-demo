pipeline {
    agent {
       label {
           label "Java"
       } 
    }

    environment {
        dockerHome = tool 'myDocker'
        mavenHome = tool 'myMaven'
        PATH = "$dockerHome/bin:$mavenHome/bin:$PATH"

    }

    stages {
        stage('Checkout') {
            steps {
                sh "mvn --version"
                sh "docker --version"            
            }
        }
        // stage('Build') {
        //     steps {
        //         sh "mvn clean install"
        //     }
        // }
        stage('Image Build and Push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    ##### INTERNAL DOCKER IMAGE BUILD & PUSH #############
                    echo "INTERNAL DOCKER IMAGE BUILD & PUSH"
                    POM_LOCAL_VERSION=$(yq e '.version.pom' gjg_restapi_backend_dev_version.yaml)
                    RELEASE_VERSION=$(yq e '.version.digit' gjg_restapi_backend_dev_version.yaml)
                    echo $POM_LOCAL_VERSION
                    echo $RELEASE_VERSION
                    POM_VERSION=$(grep -m2 '<version>' /home/jenkins-slave-01/workspace/gjg-restapi/pom.xml | tail -n1 | awk -F '>' '{ print $2 }' | awk -F '<' '{ print $1 }')                    
                    echo $POM_VERSION

                    if [ "$POM_VERSION" = "$POM_LOCAL_VERSION" ]; then
                        echo $RELEASE_VERSION
                        RELEASE_VERSION=$((RELEASE_VERSION + 1))
                        echo $RELEASE_VERSION
                        echo "pom versions are equal."
                    fi
                    '''
                }
            }
        }
        // stage ('AWS Build and Push Image'){
        //     steps {
        //         withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //             //  export AWS_PROFILE=default // this line is the first line of sh
        //             sh '''
        //             aws --version
        //             aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 461902953491.dkr.ecr.eu-west-1.amazonaws.com
        //             docker build -t gjg-restapi:latest .
        //             docker tag gjg-restapi:latest 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:latest
        //             docker push 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:latest
        //             '''
        //         }
        //     }
        // }
        // stage('Update Version and Tag') {
        //     steps {
        //         script {
        //             // read current version 
        //             pom = readMavenPom file: 'pom.xml'
        //             currentVersion = pom.version

        //             // read previous version
        //             def filename = 'version.yaml'
        //             def data = readYaml file: filename
        //             previousVersion = data.version
                    
        //             // read old tag
        //             def filetag = 'tag.yaml'
        //             def tagdata = readYaml file: filetag
        //             previousTag = tagdata.tag

        //             // compare current and previous version
        //             def currentTag = previousTag
        //             if (previousVersion == currentVersion) {
        //                 final oldTagLastDigit = previousTag.substring(previousTag.length()-1) as int
        //                 // echo "$oldTagLastDigit"
        //                 final deleted = previousTag.substring(0, previousTag.length()-1)
        //                 // echo "$deleted"
        //                 currentTag = deleted + (++oldTagLastDigit)
        //                 // echo "$currentTag"
        //             }
        //             else {
        //                 final oldTagFirstDigit = previousTag.substring(0, 1) as int
        //                 // echo "$oldTagFirstDigit"
        //                 currentTag = "" + (++oldTagFirstDigit) + ".0.1"
        //                 // echo "$currentTag"
        //             }
                    
        //             // Change old version and tag
        //             data.version = currentVersion
        //             sh "rm $filename"
        //             writeYaml file: filename, data: data

        //             tagdata.tag = currentTag
        //             sh "rm $filetag"
        //             writeYaml file: filetag, data: tagdata

        //             // set env variable to use in next script tag
        //             env.TAG = currentTag
        //         }
        //     }
        // }
        

        // stage('Build and Push Docker Image') {
        //     steps {
        //         script {
        //             // dockerImage = docker.build("meakkurt/gjg-restapi-demo")
        //             docker.withRegistry(
        //                 'https://461902953491.dkr.ecr.eu-west-1.amazonaws.com',
        //                 'ecr:eu-west-1:aws-ecr') {
        //                     def image = docker.build('gjg-restapi-demo')
        //                     image.push(env.TAG)
        //             }   
        //         }
        //     }
        // }
        
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
    post {
        always {
            echo "Build finished"
        }
        failure {
            mail to: 'furkan.akkurt@commencis.com',
            subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}
