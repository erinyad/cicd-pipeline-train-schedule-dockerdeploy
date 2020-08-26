pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker image') {
            when {
                branch 'master'
            }
            steps {
                dockerfile {
                filename 'Dockerfile'
                label 'erinyad/trainSchedule'
                registryUrl 'https://registry.hub.docker.com'
                registryCredentialsId 'docker_hub_login'
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Pass'
            }
        }
    }
}
