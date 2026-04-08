
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/one-two345/boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        // stage('Publish To Nexus') {
        //     steps {
        //       withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
        //             sh "mvn deploy"
        //         }
        //     }
        // }
        
        stage('Build & Tag Docker Image') {
            steps {
              script {
                  withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t 199900000/boardgame:latest ."
                    }
              }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html 199900000/boardgame:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
              script {
                  withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push 199900000/boardgame:latest"
                    }
              }
            }
        }
        
        stage('Deploy to Server') {
            steps {
                script {
                     sh """
                        docker pull 199900000/boardgame:latest
                        docker stop boardgame-app || true
                        docker rm boardgame-app || true
                        docker run -d --name boardgame-app -p 8080:8080 199900000/boardgame:latest
                    """
                }
            }
        }
        
    }
    
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'bela.tek.1234@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

}
