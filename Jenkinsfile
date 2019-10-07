docker login --username=sum41k --email=<hub-email>
docker build -t sum41k/train-schedule

Dockerfile:
FROM node:carbon
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]

docker run -p 8083:8080 -d sum41k/train-schedule --restart always

docker update <container_id> --restart always

Вот что нужно делать на сервере где находится дженкинс:

sudo yum -y install docker
sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
Таким образом ставится докер и даются права дженкинсу на его использование.


Настройки пайплайна с докером: 
1. Нужны 3 пользователя:
- docker_hub_login # доступы от моего аккаунта на докерхабе
- webserver_login # сервер для продакшена
- github_login доступы к репозиторию гита :
https://github.com/sum41k/cicd-pipeline-train-schedule-dockerdeploy

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
                    app = docker.build("sum41k/train-schedule")
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
                        'docker pull sum41k/train-schedule:${env.BUILD_NUMBER}\'
                        try {
                            'docker stop train-schedule'
                            'docker rm train-schedule'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        'docker run --restart always --name train-schedule -p 8083:8080 -d sum41k/train-schedule:${env.BUILD_NUMBER}\'
                    }
                }
            }
        }
    }
}
