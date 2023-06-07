pipeline {
    agent any

    stages {
        stage('Load Parameters') {
            steps {
                script {
                    env.PROPS = readProperties file: 'config.properties'
                    println "Loaded properties: ${env.PROPS}"
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
    }
}
