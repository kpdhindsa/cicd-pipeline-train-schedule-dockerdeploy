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
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("kpdhindsa00/train-schedule")
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
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p Trustwave00! ssh ubuntu@52.14.221.2 sudo docker pull kpdhindsa00/train-schedule:${env.BUILD_NUMBER}"
                        try {
                            sh "sshpass -p Trustwave00! ssh ubuntu@52.14.221.2 sudo docker stop train-schedule"
                            sh "sshpass -p Trustwave00! ssh ubuntu@52.14.221.2 sudo docker rm train-schedule"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p Trustwave00! ssh ubuntu@52.14.221.2 sudo docker run --restart always --name train-schedule -p 8080:8080 -d kpdhindsa00/train-schedule:${env.BUILD_NUMBER}"
                    }
                }
            }
        }
    }
}
