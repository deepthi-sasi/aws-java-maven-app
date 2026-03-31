#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
        [$class       : 'GitSCMSource',
         remote       : 'https://github.com/deepthi-sasi/jenkins-shared-library.git',
         credentialsID: 'ba7d5282-250d-4453-8267-b1a5fb20dbad'
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
                    dockerImagePush(env.IMAGE_NAME)
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2'
                    def shellCmd = "bash ./server-cmds.sh"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@34.253.208.5:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@34.253.208.5:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.253.208.5 ${shellCmd}"
                    }
                }
            }
        }
    }
}
