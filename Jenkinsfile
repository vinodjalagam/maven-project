pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk21'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile Server') {
            steps {
                dir('server') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Server Unit Tests') {
            steps {
                dir('server') {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit 'server/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package Server') {
            steps {
                dir('server') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Compile Webapp') {
            steps {
                dir('webapp') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Webapp Unit Tests') {
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

        stage('Package Webapp') {
            steps {
                dir('webapp') {
                    sh 'mvn package -DskipTests'
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
            archiveArtifacts artifacts: 'webapp/target/*.jar', fingerprint: true
        }
    }
}
