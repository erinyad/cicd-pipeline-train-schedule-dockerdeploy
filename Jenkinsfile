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
                script {
                    app = docker.build("erinyad/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to prod ?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull erinyad/train-schedule:$env.BUILD_NUMBER\""
                        try {
                            sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch(err) {
                            echo "Error: $err"
                        }
                        sh "sshpass -p $USERPASS -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run -d --restart always --name train-schedule -p 8080:8080 erinyad/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
