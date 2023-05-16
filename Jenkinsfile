pipeline {
    agent any

    stages {
        stage('Checkstyle') {

            agent any

            steps {
                // Use Maven checkstyle plugin to generate a code style report
                sh './mvnw checkstyle:checkstyle'
                // Save the report as a job artifact
                archiveArtifacts artifacts: 'target/site/checkstyle.html', fingerprint: true
            }
        }


        stage('Test') {

            agent any

            steps {
                // Run Maven tests
                sh './mvnw test'
            }
        }

        stage('Build') {

            agent any

            steps {
                // Build without tests using Maven
                sh './mvnw package -DskipTests'
            }
        }

        stage('Build MR image') {

            dockerImage = docker.build("rshnmrry/mr:merge-request-{{GIT_COMMIT}}")
        }

        stage('Push image') {

            withDockerRegistry([ credentialsId: "dockerhub", url: "https://hub.docker.com/repository/docker/rshnmrry/mr" ]) {

                dockerImage.push()
            }
        }

       stage('Build Main image') {

            dockerImage = docker.build("rshnmrry/main:mainline-{{GIT_COMMIT}}")
        }

        stage('Push image') {

            withDockerRegistry([ credentialsId: "dockerhub", url: "https://hub.docker.com/repository/docker/rshnmrry/main" ]) {

                dockerImage.push()
            }
        }
    }
}

