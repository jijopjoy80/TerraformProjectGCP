pipeline {
    agent any

    stages {
        stage('Load Parameters') {
            steps {
                script {
                    env.PROPS = readProperties file: 'config.properties'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def imageName = env.PROPS.getProperty('IMAGE_NAME')
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Docker Push to GCP') {
            steps {
                script {
                    def imageName = env.PROPS.getProperty('IMAGE_NAME')
                    def projectName = env.PROPS.getProperty('PROJECT_NAME')

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
    }
}
