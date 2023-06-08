pipeline {
    agent any
    
    environment {
        IMAGE_NAME_ENV = ""
        PROJECT_NAME_ENV = ""
        PATH = "/home/jenkins/google-cloud-sdk/bin:$PATH"
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
        
        stage('Install Google Cloud SDK') {
    steps {
        script {
            sh """
                rm -rf /home/jenkins/google-cloud-sdk
                curl https://sdk.cloud.google.com > install.sh
                bash install.sh --disable-prompts --install-dir=/home/jenkins
                bash -c "source /home/jenkins/google-cloud-sdk/completion.bash.inc"
                bash -c "source /home/jenkins/google-cloud-sdk/path.bash.inc"
                gcloud config set project blissful-flame-388621
                gcloud auth activate-service-account --key-file=/home/jenkins/terraform-gcp/blissful-flame-388621-5a7858ffa9ef.json
                gcloud container clusters get-credentials my-gke-cluster --region us-central1
                gcloud components install gke-gcloud-auth-plugin
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
                sh '#terraform destroy -auto-approve'
            }
        }
    }
}
