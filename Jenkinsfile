#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
        [$class       : 'GitSCMSource',
         remote       : 'git@github.com:deepthi-sasi/jenkins-shared-library.git',
         credentialsID: 'ba7d5282-250d-4453-8267-b1a5fb20dbad'
        ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'deepthisasi/demo-app:java-maven-app:1.0'
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
