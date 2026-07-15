pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                dir('server') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
