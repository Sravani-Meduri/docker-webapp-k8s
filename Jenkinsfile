pipeline {
    agent any
    tools {
        maven 'myMaven'
    }
    stages {
        stage('checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/Sravani-Meduri/docker-webapp-k8s.git'
            }
        }
        stage('Build the code using maven'){
            steps {
                sh 'mvn clean install'
            }
        }
        stage('CQA'){
            steps {
        withSonarQubeEnv(
            installationName: 'mySonar',
            credentialsId: 'sonarID'
        ) {
            sh '''
                mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                  -Dsonar.projectKey=myProject \
                  -Dsonar.projectName="myProject" \
                  -Dsonar.host.url=http://18.205.105.122:9000/ \
                  -Dsonar.token="$SONAR_AUTH_TOKEN"
            '''
        }
    }
            }
        stage('Quality Gates'){
            steps{
                waitForQualityGate abortPipeline: true, credentialsId: 'sonarID'
            }
            
        }
        
        stage('Artifacts in nexus'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'vprofile', classifier: '', file: 'target/vprofile-v2.war', type: 'war']], credentialsId: 'nexusID', groupId: 'com.visualpathit', nexusUrl: '32.197.187.146:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'myRepo', version: 'v2'
                
            }
            
        }
        stage('Docker images'){
            steps{
                sh 'docker build -t appimage .'
                sh 'docker build -t dbimage Docker-db'
            }
        }
        stage('Trivy Scan'){
            steps{
                sh 'trivy image appimage>>appimage-reports.txt'
                sh 'trivy image dbimage>>dbimage-reports.txt'
            }
        }
        stage('Docker push to registry'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh 'docker tag appimage sravanif5/k8s:app'
                        sh 'docker tag dbimage sravanif5/k8s:db'
                        sh 'docker push sravanif5/k8s:app'
                        sh 'docker push sravanif5/k8s:db'
}
                }
            }
        }
        stage('compose file'){
            steps{
                sh 'docker-compose up -d'
                sleep 20
                sh 'docker ps'
            }
        }
    }
}
    
