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
                sh "echo hi"
            }
        }
        // stage('Build') {
        //     steps {
        //         sh "mvn clean install"
        //     }
        // }
        // stage('Image Build and Push') {
        //     steps {
        //         withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //             sh '''
        //             ##### INTERNAL DOCKER IMAGE BUILD & PUSH #############
        //             echo "INTERNAL DOCKER IMAGE BUILD & PUSH"
        //             POM_LOCAL_VERSION=$(yq e '.version.pom' gjg_restapi_backend_dev_version.yaml)
        //             RELEASE_VERSION=$(yq e '.version.release' gjg_restapi_backend_dev_version.yaml)
        //             echo $POM_LOCAL_VERSION
        //             echo $RELEASE_VERSION
        //             POM_VERSION=$(grep -m2 '<version>' /home/jenkins-slave-01/workspace/gjg-restapi/pom.xml | tail -n1 | awk -F '>' '{ print $2 }' | awk -F '<' '{ print $1 }')                    
        //             echo $POM_VERSION

        //             if [ "$POM_VERSION" = "$POM_LOCAL_VERSION" ]; then
        //                 RELEASE_VERSION=$(./version.sh $RELEASE_VERSION bug) yq e -i '.version.release=env(RELEASE_VERSION)' gjg_restapi_backend_dev_version.yaml
        //                 echo "pom versions are equal."
        //             else
        //                 RELEASE_VERSION=$(./version.sh $RELEASE_VERSION major) yq e -i '.version.release=env(RELEASE_VERSION)' gjg_restapi_backend_dev_version.yaml
        //                 echo "pom versions  are not equal."
        //             fi                    
        //             '''
        //         }
        //     }
        // }
        // stage ('AWS Build and Push Image'){
        //     steps {
        //         withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //             //  export AWS_PROFILE=default // this line is the first line of sh
        //             sh '''
        //             aws --version
        //             aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 461902953491.dkr.ecr.eu-west-1.amazonaws.com
        //             IMAGE_TAG=$(yq e '.version.release' gjg_restapi_backend_dev_version.yaml)
        //             docker build -t gjg-restapi:${IMAGE_TAG} .
        //             docker tag gjg-restapi:${IMAGE_TAG} 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:${IMAGE_TAG}
        //             docker push 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:${IMAGE_TAG}
        //             '''
        //         }
        //     }
        // }
    }
    // Good when you want to clean up and etc
    // post {
    //     always {
    //         echo "Build finished"
    //     }
    //     failure {
    //         mail to: 'furkan.akkurt@commencis.com',
    //         subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
    //         body: "Something is wrong with ${env.BUILD_URL}"
    //     }
    // }
}
