pipeline {
    agent any

    stages {
        stage('Install kubectl') {
            steps {
                script {
                    sh """
                        #!/bin/bash
                        set +e
                        KUBE_VERSION=\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
                        curl -LO https://storage.googleapis.com/kubernetes-release/release/\${KUBE_VERSION}/bin/linux/amd64/kubectl
                        chmod +x kubectl
                        sudo mv kubectl /usr/local/bin/
                    """
                    sh 'kubectl version --client'
                }
            }
        }
    }
}
