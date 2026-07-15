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
        // stage('OWASP Dependency Check') {
        //     steps {
        //         dependencyCheck(
        //             odcInstallation: 'dependency-check',
        //             additionalArguments: '--scan server --format XML --format HTML',
        //             stopBuild: true
        //         )
        //     }
        // }
        
        // stage('Publish Dependency Check Report') {
        //     steps {
        //         dependencyCheckPublisher(
        //             pattern: '**/dependency-check-report.xml'
        //         )
        //     }
        // }
        stage('Trivy File System Scan') {
            steps {
                dir('server') {
                    sh '''
                        trivy fs \
                        --scanners vuln,secret,misconfig \
                        --severity HIGH,CRITICAL \
                        .
                    '''
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
    stage('Docker Build') {
        steps {
            dir('server') {
                sh 'docker build -t vinodjalagam/maven-project:${BUILD_NUMBER} .'
            }
        }
    }

    stage('Docker Login') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                '''
            }
        }
    }

    stage('Docker Push') {
        steps {
            sh 'docker push vinodjalagam/maven-project:${BUILD_NUMBER}'
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
