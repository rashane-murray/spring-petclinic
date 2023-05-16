pipeline {
    agent any

    stages {
        stage('Checkstyle') {
            steps {
                sh './mvnw checkstyle:checkstyle'
                archiveArtifacts artifacts: 'target/site/checkstyle.html', onlyIfSuccessful: true
            }
        }
        stage('Test with Maven') {
            steps {
                sh './mvnw test'
            }
        }
        stage('Build') {
            steps {
                sh './mvnw package -DskipTests'
            }
        }
        stage('Create a docker image for MR') {
            agent any
            steps {
                script {
                    def gitCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def dockerImage = docker.build("mr:${gitCommit}", "-f Dockerfile .")
                    withDockerRegistry([credentialsId: 'dockerhub', url: 'https://hub.docker.com/repository/docker/rshnmrry/mr']) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Build and push docker image for main branch') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def gitCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    def dockerImage = docker.build("main:${gitCommit}", "-f Dockerfile .")
                    withDockerRegistry([credentialsId: 'dockerhub', url: 'https://hub.docker.com/repository/docker/rshnmrry/main']) {
                        dockerImage.push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
