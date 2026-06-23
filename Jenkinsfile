pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-cred',
                    url: 'https://github.com/omprakash2929/Boardgame-Corporate-level-Pipeline.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Boardgame \
                    -Dsonar.projectKey=Boardgame \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false,
                    credentialsId: 'sonar-token'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Publish To Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-settings',
                    jdk: 'jdk17',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh 'mvn deploy'
                }
            }
        }

        stage('Build && TAG Docker Image') {
            steps {
           withDockerRegistry(credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/') {
                sh 'docker build -t omprakash2929/boardshack:latest .'
                 }
            }
        }

        stage('Docker Image Scan') {
            steps {
                sh 'trivy image --timeout 30m --format table -o trivy-image-report.html omprakash2929/boardshack:latest'
            }
        }

        stage('Push Docker Image') {
            steps {
             withDockerRegistry(credentialsId: 'dockerhub-creds', url: 'https://index.docker.io/v1/')  {
                    sh 'docker push omprakash2929/boardshack:latest'
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8s-cred',
                    namespace: 'webapps',
                    serverUrl: 'https://127.0.0.1:6443'
                ) {
                    sh 'kubectl apply -f deployment-service.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8s-cred',
                    namespace: 'webapps',
                    serverUrl: 'https://127.0.0.1:6443'
                ) {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }

    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'SUCCESS'
                def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

                def body = """
                <html>
                <body>
                    <div style="border:4px solid ${bannerColor};padding:10px;">
                        <h2>${jobName} - Build ${buildNumber}</h2>
                        <div style="background:${bannerColor};padding:10px;">
                            <h3 style="color:white;">
                                Pipeline Status: ${pipelineStatus}
                            </h3>
                        </div>
                        <p>
                            Check the <a href="${env.BUILD_URL}">
                            console output</a>.
                        </p>
                    </div>
                </body>
                </html>
                """

                emailext(
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                    body: body,
                    to: 'chauhanomprakash7206@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html,trivy-fs-report.html'
                )
            }
        }
    }
}
