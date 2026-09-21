pipeline {

    agent any

    environment {

        // Java 21
        JAVA_HOME = '/usr/lib/jvm/java-21-amazon-corretto'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        // AWS
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '193131272475'

        // ECR
        ECR_REPOSITORY = 'jdocker-java-demo'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_URI        = "${ECR_REGISTRY}/${ECR_REPOSITORY}"

        // ECS
        ECS_CLUSTER = 'java-demo-cluster'
        ECS_SERVICE = 'ecs-defination-service-o4xola2t'
        
        ECS_TASK_FAMILY = 'ecs-defination' 
        CONTAINER_NAME  = 'docker-java-demo' 

        // Docker image
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Check Java and Maven') {
            steps {
                sh '''
                    echo "========== JAVA VERSION =========="
                    java -version

                    echo "========== JAVA HOME =========="
                    echo $JAVA_HOME

                    echo "========== MAVEN VERSION =========="
                    mvn -version
                '''
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shivamingale500-ui/Java-Application-CI-Pipeline-.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                    echo "========== MAVEN BUILD =========="

                    mvn clean package -DskipTests
                '''
            }
        }
        
        stage('Docker Build') {
            steps {
                sh """
                    echo "========== DOCKER BUILD =========="

                    docker build --no-cache -t ${ECR_URI}:${IMAGE_TAG} .
                """
            }
        }

        stage('ECR Login') {
            steps {
                sh """
                    echo "========== ECR LOGIN =========="

                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Docker Tag') {
            steps {
                sh """
                    echo "========== DOCKER TAG =========="

                    docker tag ${ECR_URI}:${IMAGE_TAG} ${ECR_URI}:latest
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                    echo "========== PUSH IMAGE TO ECR =========="

                    docker push ${ECR_URI}:${IMAGE_TAG}
                    docker push ${ECR_URI}:latest
                """
            }
        }

             stage('Deploy to ECS') {
            steps {
                sh """
                    echo "========== ECS DEPLOYMENT =========="
                    
                    aws ecs describe-task-definition --task-definition ${ECS_TASK_FAMILY} --region ${AWS_REGION} --query taskDefinition > task-def.json
                    
                    jq '. | {family, containerDefinitions, volumes, networkMode, placementConstraints, requiresCompatibilities, cpu, memory, taskRoleArn, executionRoleArn}' task-def.json > cleaned-task-def.json
                    
                    sed -i 's|"image": ".*"|"image": "${ECR_URI}:${IMAGE_TAG}"|g' cleaned-task-def.json
                    
                    aws ecs register-task-definition --cli-input-json file://cleaned-task-def.json --region ${AWS_REGION} > registered-task.json

                    aws ecs update-service \
                    --cluster ${ECS_CLUSTER} \
                    --service ${ECS_SERVICE} \
                    --task-definition ${ECS_TASK_FAMILY} \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                    
                    rm -f task-def.json cleaned-task-def.json registered-task.json
                """
            }
        }

        
    }

    post {
        
        success {
            echo '======================================'
            echo ' CI/CD PIPELINE SUCCESSFUL'
            echo ' Java 21 Build Successful'
            echo ' Docker Image Built'
            echo ' Image Pushed to ECR'
            echo ' ECS Deployment Triggered with Dynamic Revision Tracking'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo ' CI/CD PIPELINE FAILED'
            echo ' Check the console output'
            echo '======================================'
        }
    }
}
