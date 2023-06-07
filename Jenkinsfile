pipeline {
    agent any
    
        environment {
        IMAGE_NAME_ENV = ""
        PROJECT_NAME_ENV = ""
    }
    
    stages {
        stage('Load and Set Parameters') {
            steps {
                script {
                    def propsMap = readProperties file: 'config.properties'
                    env.IMAGE_NAME_ENV = propsMap['IMAGE_NAME']
                    env.PROJECT_NAME_ENV = propsMap['PROJECT_NAME']
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def propsMap = readProperties file: 'config.properties'
                    def imageName = propsMap['IMAGE_NAME']
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Docker Push to GCP') {
            steps {
                script {
                    def propsMap = readProperties file: 'config.properties'
                    def imageName = propsMap['IMAGE_NAME']
                    def projectName = propsMap['PROJECT_NAME']

                    sh """
                        yes | /home/jenkins/terraform-gcp/google-cloud-sdk/bin/gcloud auth configure-docker --quiet
                        docker tag ${imageName}:latest gcr.io/${projectName}/${imageName}:latest
                        docker push gcr.io/${projectName}/${imageName}:latest
                    """
                }
            }
        }

        stage('GKE Build') {
            steps {
                script {
                    sh """
                        terraform init
                        terraform apply -auto-approve
                    """
                }
            }
        }
        
        stage('Install kubectl') {
    steps {
        script {
            sh """
                curl -LO "https://storage.googleapis.com/kubernetes-release/release/\\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x ./kubectl
                sudo mv ./kubectl /usr/local/bin/kubectl
            """
        }
    }
}   
        stage('Kubernetes Deployment') {
            steps {
                script {
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                    sh "kubectl get service my-nginx-service"
                }
            }
        }
        
        stage('Wait') {
            steps {
                sh 'sleep 180'
            }
        }
        
        stage('Terraform Destroy') {
            steps {
                    sh 'terraform destroy -auto-approve'
            }
        }
    }
}
