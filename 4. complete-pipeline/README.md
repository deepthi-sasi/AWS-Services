## Demo Project - Complete CI/CD Pipeline

### Topics of the Demo Project

Complete the CI/CD Pipeline (Docker-Compose, Dynamic versioning)

### Technologies Used
* AWS
* Jenkins 
* Docker 
* Linux 
* Git 
* Java 
* Maven 
* Docker Hub

### Project Description
CI step: Increment version
CI step: Build artifact for Java Maven application
CI step: Build and push Docker image to Docker Hub
CD step: Deploy new application version with Docker Compose
CD step: Commit the version update

Steps to implement the complete CI/CD Pipeline
Step 1: Compared to the pipeline of the previous demo project 3 only the first stage (incrementing the version) and the last stage (committing the version update) are missing. So we copy these two stages from the final pipeline in module 08 (Build Automation & CI/CD with Jenkins). This results in the following Jenkinsfile for the complete pipeline:

```
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
```


The shell script, called in the stage "Deploy Application" just contains the following lines:

```
#!/usr/bin/env/ bash
export IMAGE_TAG=$1
docker-compose -f docker-compose.yaml up -d
echo "successfully started the container using docker-compose"```