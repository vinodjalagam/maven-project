pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven'
    }
    environment {
        IMAGE = "vinodjalagam/maven-project"
}

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Set Image Tag') {
            steps {
                script {
                    def DATE = sh(
                        script: 'date +%d%m%Y',
                        returnStdout: true
                    ).trim()

                    env.IMAGE = "${IMAGE_NAME}:spring-${BUILD_NUMBER}-${DATE}"

                    echo "Docker Image: ${env.IMAGE}"
                }
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
   
        stage('Docker Build') {
            steps {
                dir('server') {
                    sh 'docker build -t $IMAGE .'
                }
            }
        }
        // stage('Trivy Image Scan') {
        //     steps {
        //         sh '''
        //         trivy image \
        //         --severity HIGH,CRITICAL \
        //         vinodjalagam/maven-project:${BUILD_NUMBER}
        //         '''
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

                    echo "Generating Full Trivy Report..."
        
                    # Scan the Docker image and generate HTML report
                    trivy image \
                        --format template \
                        --template "@$HOME/.trivy/html.tpl" \
                        -o ${WORKSPACE}/reports/trivy-image-report.html \
                        $IMAGE
                        
                    echo "Generating HIGH & CRITICAL Report..."

                    trivy image \
                        --severity HIGH,CRITICAL \
                        --format template \
                        --template "@$HOME/.trivy/html.tpl" \
                        -o ${WORKSPACE}/reports/trivy-report-high-critical.html \
                        $IMAGE
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
                sh 'docker push $IMAGE'
            }
        }
     }

    post {

        always {
            archiveArtifacts artifacts: 'server/target/*.jar'
            archiveArtifacts artifacts: 'reports/*.html', fingerprint: true
                    
            emailext(
                subject: "Build #${BUILD_NUMBER} - Trivy Security Report",
                body: """
                <h2>Jenkins Build: ${JOB_NAME}</h2>
                <h2>Docker Image: ${IMAGE}</h2>
    
                <p><b>Build Number:</b> ${BUILD_NUMBER}</p>
    
                <p><b>Status:</b> ${currentBuild.currentResult}</p>
    
                <p>Attached are the Trivy Image Scan Reports.</p>
    
                <ul>
                    <li>Full Vulnerability Report</li>
                    <li>High & Critical Vulnerability Report</li>
                </ul>
    
                Regards,<br>
                Jenkins CI/CD
                """,
                mimeType: 'text/html',
                to: 'vinodjalagam477@gmail.com',
                attachmentsPattern: 'reports/*.html'
                )

       }

        success {
            echo "Pipeline Successful"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
