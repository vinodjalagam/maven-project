pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven'
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
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Unit Tests') {
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

        stage('JaCoCo Coverage') {
            steps {
                dir('server') {
                    sh 'mvn verify'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('server') {

                    withSonarQubeEnv('sonarqube') {

                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=maven-project \
                        -Dsonar.projectName=maven-project
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                dir('server') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
    }

    post {

        always {
            archiveArtifacts artifacts: 'server/target/*.jar'
        }

        success {
            echo "Pipeline Successful"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
