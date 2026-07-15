pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk17'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                dir('server') {
                    sh 'mvn -DskipTests clean package'
                }
            }
        }
        

        stage('Unit Tests') {
            steps {
                dir('server') {
                    sh 'mvn test'
                }
            }

        stage('Compile') {
            steps {
                dir('webapp') {
                    sh 'mvn -DskipTests clean package'
                }
            }
        }
        stage('Unit Tests') {
            steps {
                dir('webapp') {
                    sh 'mvn test'
                }
            }

            post {
                always {
                    junit 'webapp/target/surefire-reports/*.xml'
                }
            }
        }

    }

    post {

        success {
            echo "✅ Build Successful"
        }

        failure {
            echo "❌ Build Failed"
        }

        always {
            archiveArtifacts artifacts: 'server/target/*.jar', fingerprint: true
        }
        always {
            archiveArtifacts artifacts: 'webapp/target/*.jar', fingerprint: true
        }
    }
}
