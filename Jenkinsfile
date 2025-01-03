pipeline {
    agent any

    environment {
        GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        DOCKER_CREDS = credentials('docker_key')
    }

    stages {
        stage ('Checkstyle') {
            steps {
                sh 'mvn checkstyle:checkstyle'
                archiveArtifacts artifacts: 'target/checkstyle-report.xml', allowEmptyArchive: true
                sh 'echo Checkstyle is completed!'
            }
        }
        stage ('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: false
            }
        }
        stage ('Creating Docker image') {
            steps {
                script {
                    def dockerRepo = (env.BRANCH_NAME == 'main') ? 'vkarpenko02/main' : 'vkarpenko02/mr'
                    def dockerTag = "${dockerRepo}:${GIT_COMMIT}"
                    sh "docker build -t ${dockerTag} ."
                    withCredentials([usernamePassword(credentialsId: 'docker_key', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                        sh """
                            echo ${DOCKER_HUB_PASS} | docker login -u ${DOCKER_HUB_USER} --password-stdin
                            docker push ${dockerTag}
                        """
                    }
                }
            }
        }
    }
}