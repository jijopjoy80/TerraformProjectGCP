pipeline {
    agent any
    
    environment {
        IMAGE_NAME_ENV = ""
        PROJECT_NAME_ENV = ""
        CRED_FILE = ""
        PATH = "/home/jenkins/google-cloud-sdk/bin:$PATH"
    }
    
    stages {
        stage('Load and Set Parameters') {
            steps {
                script {
                    def propsMap = readProperties file: 'config.properties'
                    env.IMAGE_NAME_ENV = propsMap['IMAGE_NAME']
                    env.PROJECT_NAME_ENV = propsMap['PROJECT_NAME']
                    env.CRED_FILE = propsMap['CRED_FILE']
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
                    def credentialFile = propsMap['CRED_FILE']

                    sh """
                        rm -rf /home/jenkins/google-cloud-sdk
                        curl https://sdk.cloud.google.com > install.sh
                        bash install.sh --disable-prompts --install-dir=/home/jenkins
                        bash -c "source /home/jenkins/google-cloud-sdk/completion.bash.inc"
                        bash -c "source /home/jenkins/google-cloud-sdk/path.bash.inc"
                        gcloud config set project ${projectName}
                        gcloud auth activate-service-account --key-file=/home/jenkins/${credentialFile}
                        yes | /home/jenkins/google-cloud-sdk/bin/gcloud auth configure-docker --quiet
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
        
        stage('Deploy Kubernetes Cluster') {
        steps {
            script {
                def propsMap = readProperties file: 'config.properties'
                 def projectName = propsMap['PROJECT_NAME']
                 def credentialFile = propsMap['CRED_FILE']
            
                    sh """
                        gcloud config set project ${projectName}
                        gcloud auth activate-service-account --key-file=/home/jenkins/${credentialFile}
                        gcloud container clusters get-credentials my-gke-cluster --region us-central1
                        gcloud components install gke-gcloud-auth-plugin
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        sleep 180
                        kubectl get service my-nginx-service
                        curl http://\$(kubectl get service my-nginx-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}') > gcp_output_page.txt
                        cat gcp_output_page.txt
                        kubectl delete deployments --all
                        kubectl delete service my-nginx-service
                    """
                }
            }
        }

        stage('Terraform Destroy') {
            steps {
                //sh 'terraform destroy -auto-approve'
            }
        }
    }
}
