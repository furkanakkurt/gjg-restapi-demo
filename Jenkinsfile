pipeline {
    agent {
       label {
           label "Java"
       } 
    }

    stages {
        stage('Checkout') {
            steps {
                sh "mvn --version"
                sh "docker --version" 
                sh "java -version"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Image Build and Push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    ##### INTERNAL DOCKER IMAGE BUILD & PUSH #############
                    echo "INTERNAL DOCKER IMAGE BUILD & PUSH"
                    POM_LOCAL_VERSION=$(yq e '.version.pom' gjg_restapi_backend_dev_version.yaml)
                    RELEASE_VERSION=$(yq e '.version.release' gjg_restapi_backend_dev_version.yaml)
                    echo $POM_LOCAL_VERSION
                    echo $RELEASE_VERSION
                    POM_VERSION=$(grep -m2 '<version>' /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/pom.xml | tail -n1 | awk -F '>' '{ print $2 }' | awk -F '<' '{ print $1 }')                    
                    echo $POM_VERSION

                    if [ "$POM_VERSION" = "$POM_LOCAL_VERSION" ]; then
                        RELEASE_VERSION=$(./version.sh $RELEASE_VERSION bug) yq e -i '.version.release=env(RELEASE_VERSION)' gjg_restapi_backend_dev_version.yaml
                        echo "pom versions are equal."
                    else
                        RELEASE_VERSION=$(./version.sh $RELEASE_VERSION major) yq e -i '.version.release=env(RELEASE_VERSION)' gjg_restapi_backend_dev_version.yaml
                        echo "pom versions  are not equal."
                    fi                    
                    '''
                }
            }
        }
        stage ('AWS Build and Push Image'){
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    //  export AWS_PROFILE=default // this line is the first line of sh
                    sh '''
                    aws --version
                    aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 461902953491.dkr.ecr.eu-west-1.amazonaws.com
                    IMAGE_TAG=$(yq e '.version.release' gjg_restapi_backend_dev_version.yaml)
                    docker build -t gjg-restapi:${IMAGE_TAG} .
                    docker tag gjg-restapi:${IMAGE_TAG} 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:${IMAGE_TAG}
                    docker push 461902953491.dkr.ecr.eu-west-1.amazonaws.com/ecr-devops-furkan:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage ('Deploy to ECS'){
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'furkan_terraform-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                    POM_VERSION=$(yq e '.version.pom' gjg_restapi_backend_dev_version.yaml)
                    RELEASE_NUMBER=$(yq e '.version.release' gjg_restapi_backend_dev_version.yaml)
                    
                    echo "POM_VERSION: $POM_VERSION"
                    echo "RELEASE_NUMBER: $RELEASE_NUMBER"

                    NAME=gjg-restapi
                    REGION=eu-west-1
                    REPOSITORY_NAME=ecr-devops-furkan
                    CLUSTER=gjg-restapi-cluster-furkan
                    FAMILY=gjg-restapi-furkan
                    SERVICE_NAME=devops-furkan-ecs-service
                    
                    #Store the repositoryUri as a variable
                    REPOSITORY_URI=`aws ecr describe-repositories --repository-names ${REPOSITORY_NAME} --region ${REGION} | jq .repositories[].repositoryUri | tr -d '"'`
                    echo "REPOSITORY_URI: $REPOSITORY_URI"

                    #Replace the build number and respository URI placeholders with the constants above
                    sed -e "s;%TAG%;${RELEASE_NUMBER};g" -e "s;%REPOSITORY_URI%;${REPOSITORY_URI};g" /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/jsons/taskdef-dev.json > /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/jsons/releases/${NAME}-v_${RELEASE_NUMBER}.json
                    
                    #Register the task definition in the repository
                    aws ecs register-task-definition --family ${FAMILY} --cli-input-json file:///home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/jsons/releases/${NAME}-v_${RELEASE_NUMBER}.json --region ${REGION}
                    
                    SERVICES=`aws ecs describe-services --services ${SERVICE_NAME} --cluster ${CLUSTER} --region ${REGION} | jq .failures[]`
                    #Get latest revision
                    REVISION=`aws ecs describe-task-definition --task-definition ${FAMILY}--region ${REGION} | jq .taskDefinition.revision`
                    
                    #Create or update service
                    DESIRED_COUNT=`aws ecs describe-services --services ${SERVICE_NAME} --cluster ${CLUSTER} --region ${REGION} | jq .services[].desiredCount`
                    sed -e "s;%FAMILY%;${FAMILY};g" -e "s;%REVISION%;${REVISION};g" /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/yamls/appspec-dev.yaml > /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/yamls/releases/${NAME}-v_${RELEASE_NUMBER}.yaml
                   
                    aws s3 cp /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/yamls/releases/${NAME}-v_${RELEASE_NUMBER}.yaml s3://gjg-restapi-backend-dev/appspec.yaml
                    cd /home/jenkins-slave-01/workspace/GJG_RESTAPI_BACKEND_DEV/jsons/
                    DEPLOYMENT_ID=$(aws deploy create-deployment --cli-input-json file:///create-deployment-dev.json --region ${REGION} | jq -r '.deploymentId')
                    aws deploy wait deployment-successful --region ${REGION} --deployment-id $DEPLOYMENT_ID
                    
                    '''
                }
            }
        }
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
    // //     }
    // }
}
