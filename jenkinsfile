// Trigger the pipeline when a push happens on GitHub
properties([
    pipelineTriggers([githubPush()])
])

pipeline {

    environment {
        // Docker repository
        dockerRepo = "daouiwalid/jenkins-hello-world"

        // Jenkins credentials ID for Docker Hub authentication
        dockerCredentials = 'DockerHub'

        // Tags for versioned and latest images
        dockerImageVersioned = ""
        dockerImageLatest = ""
    }

    // Run the pipeline on any available Jenkins agent
    agent any

    stages {

        // --------------------------------------------------
        // STAGE 1: CHECKOUT CODE FROM YOUR GITHUB REPO
        // --------------------------------------------------
        stage('Checkout SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']], // adjust if your branch is named 'master'
                    userRemoteConfigs: [[
                        url: 'https://github.com/DaouiWalid/JenkinsHelloWorld.git'
                    ]]
                ])
            }
        }

        // --------------------------------------------------
        // STAGE 2: BUILD DOCKER IMAGE
        // --------------------------------------------------
        stage("Build Docker Image") {
            steps {
                script {
                    // Pull the official hello-world image and retag it
                    sh "docker pull hello-world"

                    // Tag it for your Docker Hub repository
                    sh "docker tag hello-world ${dockerRepo}:${BUILD_NUMBER}"
                    sh "docker tag hello-world ${dockerRepo}:latest"
                }
            }
        }

        // --------------------------------------------------
        // STAGE 3: PUSH IMAGE TO DOCKER HUB
        // --------------------------------------------------
        stage("Push to Docker Hub") {
            steps {
                script {
                    // Authenticate with Docker Hub using Jenkins credentials
                    docker.withRegistry('', dockerCredentials) {
                        sh "docker push ${dockerRepo}:${BUILD_NUMBER}"
                        sh "docker push ${dockerRepo}:latest"
                    }
                }
            }
        }

        // --------------------------------------------------
        // STAGE 4: CLEANUP LOCAL IMAGES
        // --------------------------------------------------
        stage('Cleanup') {
            steps {
                sh "docker rmi ${dockerRepo}:${BUILD_NUMBER} || true"
                sh "docker rmi ${dockerRepo}:latest || true"
            }
        }
    }

    // --------------------------------------------------
    // POST-BUILD ACTIONS
    // --------------------------------------------------
    post {
        always {
            deleteDir() // clean up Jenkins workspace
        }
    }
}
