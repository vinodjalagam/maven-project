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
        // stage('Trivy File System Scan') {
        //     steps {
        //         dir('server') {
        //             sh '''
        //                 trivy fs \
        //                 --scanners vuln,secret,misconfig \
        //                 --severity HIGH,CRITICAL \
        //                 .
        //             '''
        //         }
        //     }
        // }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                    mkdir -p ${WORKSPACE}/reports
                    mkdir -p ~/.trivy
        
                    # Download HTML template if it doesn't exist
                    if [ ! -f ~/.trivy/html.tpl ]; then
                        wget -q -O ~/.trivy/html.tpl \
                        https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
                    fi
        
                    # Scan the Docker image and generate HTML report
                    trivy image \
                        --format template \
                        --template "@$HOME/.trivy/html.tpl" \
                        -o ${WORKSPACE}/reports/trivy-image-report.html \
                        vinodjalagam/maven-project:${BUILD_NUMBER}
                '''
            }
        }
        stage('Package') {
            steps {
                dir('server') {
                    sh 'mvn package -DskipTests'
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
        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image \
                --severity HIGH,CRITICAL \
                vinodjalagam/maven-project:${BUILD_NUMBER}
                '''
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
     }

    post {

        always {
            archiveArtifacts artifacts: 'server/target/*.jar'
            archiveArtifacts artifacts: 'reports/trivy-image-report.html', fingerprint: true

        }

        success {
            echo "Pipeline Successful"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
