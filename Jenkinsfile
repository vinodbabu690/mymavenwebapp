pipeline {
    agent any

    tools {
        maven 'maven3.8.2'
    }

    triggers {
        pollSCM('* * * * *')  // polls every minute
    }

    options {
        timestamps()
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '', 
                artifactNumToKeepStr: '5', 
                daysToKeepStr: '', 
                numToKeepStr: '5'
            )
        )
    }

    environment {
        DOCKER_IMAGE = "yourdockerhubusername/maven-web-app:${env.BUILD_NUMBER}"
        GITHUB_CREDENTIALS_ID = "github-credentials"        // Update with your Jenkins GitHub credentials ID
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials"   // Update with your Jenkins Docker credentials ID
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'development', 
                    credentialsId: "${GITHUB_CREDENTIALS_ID}", 
                    url: 'https://github.com/devopstrainingblr/maven-web-application-1.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Clean Up Docker Image') {
            steps {
                script {
                    echo "Removing local Docker image..."
                    docker.image("${DOCKER_IMAGE}").remove()
                }
            }
        }

        /*
        stage('Execute SonarQube Analysis') {
            steps {
                sh "mvn clean sonar:sonar"
            }
        }

        stage('Upload Artifacts to Nexus') {
            steps {
                sh "mvn clean deploy"
            }
        }

        stage('Deploy App into Tomcat') {
            steps {
                sshagent(['bfe1b3c1-c29b-4a4d-b97a-c068b7748cd0']) {
                    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@35.154.190.162:/opt/apache-tomcat-9.0.50/webapps/"
                }
            }
        }
        */
    }

    post {
        success {
            emailext to: 'devopstrainingblr@gmail.com,mithuntechnologies@yahoo.com',
                     subject: "Pipeline Build #${env.BUILD_NUMBER} - Success",
                     body: "Pipeline Build #${env.BUILD_NUMBER} completed successfully.",
                     replyTo: 'devopstrainingblr@gmail.com'
        }

        failure {
            emailext to: 'devopstrainingblr@gmail.com,mithuntechnologies@yahoo.com',
                     subject: "Pipeline Build #${env.BUILD_NUMBER} - Failed",
                     body: "Pipeline Build #${env.BUILD_NUMBER} failed. Please check the console logs.",
                     replyTo: 'devopstrainingblr@gmail.com'
        }
    }
}

