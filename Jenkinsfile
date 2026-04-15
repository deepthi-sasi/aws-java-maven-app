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
            DOCKER_REPO_SERVER = '084375549310.dkr.ecr.eu-west-1.amazonaws.com'
            DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
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
                    env.IMAGE_NAME = "$DOCKER_REPO:java-maven-app-$version-$BUILD_NUMBER"
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
                    withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        //sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        echo "${USER}"
                        buildImage(env.IMAGE_NAME)
                        dockerLoginInToHost($USER, $PASS, env.DOCKER_REPO_SERVER)
                        dockerImagePush(env.IMAGE_NAME)
                    }
                }
            }
        }

        stage("deploy") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
                APP_NAME = 'java-maven-app'
            }

            steps {
                script {
                    echo 'deploying docker image'
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        }

        stage('Commit Version Update') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'githubtoken', variable: "credentials")]) {

                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git remote set-url origin https://${credentials}@github.com/deepthi-sasi/aws-java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "jenkins: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }

    }
}
