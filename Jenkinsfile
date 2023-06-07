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
                        #terraform apply -auto-approve
                    """
                }
            }
        }
    }
}
