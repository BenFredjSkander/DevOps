def projectDockerImage
pipeline {
    agent any

    environment {
        dockerImageName = "devops-project"
        dockerUsername = "skander007"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven "maven3"
        jdk "jdk17"
    }


    stages {
        stage('Github checkout'){
            steps{
                git branch: 'skander-benfredj', credentialsId: '9dd9be9c-8871-4707-815d-9c56325c13b1', url: 'https://github.com/BenFredjSkander/DevOps'
            }
        }

        stage('Compile Backend') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
                    sh '''  mvn sonar:sonar -Dsonar.projectKey=DevopsProject'''
            }}
        }


        stage('Nexus publish'){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }


        stage('Build docker images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        projectDockerImage = docker.build dockerUsername+"/"+dockerImageName + ":latest", " ."
                    }
                        // to delete old images with none tag
                        // sh "docker images -a | grep none | awk '{ print \$3}' | xargs docker rmi --force"
                }
            }
        }

        stage('Docker image scan'){
            steps{
                script{
                    sh '''trivy image --skip-db-update --severity MEDIUM,HIGH,CRITICAL --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o docker-report-trivy.html ${dockerUsername}/${dockerImageName}:latest'''
                }
            }
        }

        stage('Docker push images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        projectDockerImage.push()
                    }
                }
            }
        }

        stage('Run docker compose stack'){
            steps{
                script{
                    sh "docker compose up -d"
                }
            }
        }

        stage('Monitor app'){
            steps{
                script{
                    sh "docker container start prometheus"
                    sh "docker container start grafana"
                }
            }
        }
    }

        post{
            always{
                script{
                        def pipelineStatus = currentBuild.result ?: "UNKNOWN"
                        def bannerColor = pipelineStatus.toUpperCase() == "SUCCESS" ? "#28a745" : "#dc3545"
                        def jobName = env.JOB_NAME
                        def buildNumber = env.BUILD_NUMBER
                        def htmlContent = """
                            <html>
                                <body>
                                    <div style="padding: 10px; color: white; background-color: ${bannerColor}; text-align: center; font-size: 24px;">
                                        ${pipelineStatus}
                                    </div>
                                    <p>Hello,</p>
                                    <p>The Jenkins job has completed with status: <strong>${pipelineStatus}</strong>.</p>
                                    <p>Check the details on your Jenkins server.</p>
                                </body>
                            </html>
                        """

                        emailext (
                            subject: "${jobName} - ${buildNumber} : ${pipelineStatus}",
                            body: htmlContent,
                            mimeType: 'text/html',
                            to: PERSONAL_EMAIL,
                            attachmentsPattern: '*-report-trivy.html'
                        )
                }
            }
        }
}