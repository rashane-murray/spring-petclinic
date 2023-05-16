pipeline {
    agent any

    stages {
        stage('Checkstyle') {
            agent {
                label 'checkstyle'
            }
            steps {
                // Use Maven checkstyle plugin to generate a code style report
                sh './mvnw checkstyle:checkstyle'
                // Save the report as a job artifact
                archiveArtifacts artifacts: 'target/site/checkstyle.html', fingerprint: true
            }
        }

        stage('Test') {
            agent {
                label 'maven'
             }
            steps {
                // Run Maven tests
                sh './mvnw test'
            }
        }

        stage('Build') {
            agent {
                label 'maven'
            }
            steps {
                // Build without tests using Maven
                sh './mvnw package -DskipTests'
            }
        }

        stage('Create Docker Image') {
            agent {
                label 'docker'
            }
            steps {
                // Build a Docker image using the Dockerfile in the spring-petclinic repo
                sh 'docker build -t mr:{{GIT_COMMIT}} .'
                // Tag the image using GIT_COMMIT (short)
                sh 'docker tag mr:{{GIT_COMMIT}} mr:{{GIT_COMMIT}}-short'
                // Push the image to the "mr" repository on Docker Hub
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                    sh 'docker push mr:{{GIT_COMMIT}}-short'
                }
            }
        }
    }

// Pipeline for the main branch
pipeline {
    agent any

    stages {
        stage('Create Docker Image') {
            agent {
                label 'docker'
            }
            steps {
                // Build a Docker image using the Dockerfile in the spring-petclinic repo
                sh 'docker build -t main:{{GIT_COMMIT}} .'
                // Push the image to the "main" repository on Docker Hub
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                    sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                    sh 'docker push main:{{GIT_COMMIT}}'
                }
            }
        }
    }
