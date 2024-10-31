pipeline {
    agent any

    environment {
       dockerImageTag = "skander007/devops-project"
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

        stage('Trivy File Scan') {
            steps {
                sh "trivy filesystem --format table -o file-report-trivy.html ."

            }
        }

        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=DevopsProject -Dsonar.java.binaries=**/classes/**'''
            }}
        }


        stage('Nexus publish'){
            steps{
                dir('back'){
                    withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        sh 'mvn deploy'
                    }
                }
            }
        }


        stage('Build docker images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker build -t ${dockerImageTag}:latest ."
                    }
                        // to delete old images with none tag
                        // sh "docker images -a | grep none | awk '{ print \$3}' | xargs docker rmi --force"
                }
            }
        }

        stage('Docker image scan'){
            steps{
                script{
                    sh "trivy image --format table -o file-report-trivy.html ${dockerImageTag}:latest"
                }
            }
        }

        stage('Docker push images'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                        sh "docker push ${dockerImageTag}:latest"
                    }
                }
            }
        }
}

