pipeline {
    agent any
    environment {
        registry = "654654178716.dkr.ecr.us-east-1.amazonaws.com/ecs-jenkins"
        clusterName = "my-cluster"
        serviceName = "my-service"
        taskDefinitionName = "my-task-definition"
        ecsRegion = "us-east-1"
    }
   
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/uday912/Ecs-project-3.git'
            }
        }
        
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }
        
        stage('Pushing to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_cred', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh 'aws ecr get-login-password --region $ecsRegion | docker login --username AWS --password-stdin $registry'
                        sh 'docker push $registry'
                    }
                }
            }
        }
        
        stage('Stop Previous Containers') {
            steps {
                sh 'docker ps -f name=myContainer -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -f name=myContainer -q | xargs -r docker container rm'
            }
        }
        
        stage('Docker Run') {
            steps {
                script {
                    sh 'docker run -d -p 80:80 --rm --name myContainer $registry:latest'
                }
            }
        }
        
        stage('Create ECS Task Definition') {
            steps {
                script {
                    def taskDefinition = """
                    {
                        "family": "$taskDefinitionName",
                        "containerDefinitions": [
                            {
                                "name": "my-container",
                                "image": "$registry:latest",
                                "essential": true,
                                "memory": 512,
                                "cpu": 256,
                                "portMappings": [
                                    {
                                        "containerPort": 80,
                                        "hostPort": 80
                                    }
                                ]
                            }
                        ]
                    }
                    """
                    writeFile file: 'task-definition.json', text: taskDefinition
                    sh 'aws ecs register-task-definition --cli-input-json file://task-definition.json --region $ecsRegion'
                }
            }
        }
        
        stage('Create ECS Cluster') {
            steps {
                script {
                    sh """
                    aws ecs create-cluster --cluster-name $clusterName --region $ecsRegion
                    """
                }
            }
        }
        
        stage('Create or Update ECS Service') {
            steps {
                script {
                    sh """
                    aws ecs create-service --cluster $clusterName --service-name $serviceName --task-definition $taskDefinitionName --desired-count 1 --region $ecsRegion
                    """
                }
            }
        }
    }
}
