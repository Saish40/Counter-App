pipeline {
    agent any
    stages{
        stage('GIT checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Saish40/Counter-App.git'
            }
        }
        stage('Unit test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Static code analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-api') {
                        sh 'mvn clean package sonar:sonar'   
                }
                }
            }
        }
        stage('Quality gate') {
            steps {
                script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage('Upload WAR to nexus ') {
            steps {
                script {
                     nexusArtifactUploader artifacts:
                      [
                        [
                            artifactId: 'springboot', 
                            classifier: '',
                            file: 'target/Uber.jar', 
                            type: 'jar']
                            ], 
                            credentialsId: 'nexus-api', 
                            groupId: 'com.example', 
                            nexusUrl: '52.3.247.229:8081', 
                            nexusVersion: 'nexus3', 
                            protocol: 'http', 
                            repository: 'demoapp-release', 
                            version: "1.0.2"
                }
            }
        }
         stage('Docker Image build') {
            steps {
                script {
                    sh 'docker build -t $JOBNAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOBNAME:v1.$BUILD_ID saish69/$JOBNAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOBNAME:v1.$BUILD_ID saish69/$JOBNAME:latest'
                }
            }
        }
        stage('Docker login and push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerpwd', usernameVariable: 'dockerusr')]) {
                sh "docker login -u ${env.dockerusr} -p ${env.dockerpwd}"
                }
                sh 'docker push $JOBNAME:v1.$BUILD_ID saish69/$JOBNAME:v1.$BUILD_ID'
                sh 'docker push $JOBNAME:v1.$BUILD_ID saish69/$JOBNAME:latest'
            }
        }
    }
}
