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
                    sh "docker build -t ${env.PROPS.IMAGE_NAME} ."
            }
        }

        stage('Docker Push to GCP') {
            steps {
                    script {
                        sh """
                            yes | /home/jenkins/terraform-gcp/google-cloud-sdk/bin/gcloud auth configure-docker --quiet
                            docker tag ${env.PROPS.IMAGE_NAME}:latest gcr.io/blissful-flame-388621/${env.PROPS.IMAGE_NAME}:latest
                            docker push gcr.io/blissful-flame-388621/${env.PROPS.IMAGE_NAME}:latest
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
