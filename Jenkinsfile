#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
        [$class       : 'GitSCMSource',
         remote       : 'https://gitlab.com/twn-devops-bootcamp/latest/09-aws/jenkins-shared-library.git',
         credentialsID: 'gitlab-credentials'
        ]
)

pipeline {
    agent any
    tools {
        maven 'maven3.9'
    }
    environment {
        IMAGE_NAME = 'deepthisasi/demo-app:java-maven-app-1.0'
    }

    stages {
        stage("build app") {
            steps {
                script {
                    echo "Building the application..."
                    buildJar()
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    def dockerCmd = 'docker run -p 3080:3080 -d ${IMAGE_NAME}'
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.253.208.5 ${dockerCmd}"
                    }
                }
            }
        }
    }
}
