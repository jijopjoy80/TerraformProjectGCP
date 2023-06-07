stage('Load Parameters') {
    steps {
        script {
            env.PROPS = readProperties file: 'config.properties'
            println "Loaded properties: ${env.PROPS}"
        }
    }
}

