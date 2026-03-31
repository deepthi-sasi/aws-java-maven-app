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

    stages {
        stage("Increment Version") {
            steps {
                script {
                    echo 'incrementing the bugfix version of the application...'
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                     versions:commit'

                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "deepthisasi/demo-app:java-maven-app-$version-$BUILD_NUMBER"
                }
            }
        }

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

                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@34.253.208.5"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        }

        stage('Commit Version Update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'ba7d5282-250d-4453-8267-b1a5fb20dbad', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/deepthi-sasi/aws-java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "jenkins: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }

    }
}
