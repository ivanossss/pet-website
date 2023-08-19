pipeline {
    agent any

    stages {
        stage('Клонирование репозитория') {
            steps {
                // Клонирование вашего репозитория
                sh 'git clone https://github.com/ivanossss/pet-website.git'
            }
        }

        stage('Сборка и запуск Docker контейнера') {
            steps {
                script {
                // Запуск контейнера с монтированием директории
                sh 'docker run -d -p 80:80 -v ./pet-website:/usr/share/nginx/html --name nginx-container nginx:latest'
                }
            }
        }

        stage('Проверка') {
            steps {
                // Выполнение curl для проверки работоспособности
                sh 'curl http://localhost'
                // Заодно настроим aws cli
                sh 'aws configure set region eu-north-1'
            }
        }
        
    
        stage('Получение списка IP-адресов') {
            steps {
                script {
                    // Сохранение IP-адресов в переменную
                    def remoteIPs = sh(script: 'aws ec2 describe-instances --filters "Name=tag:Name,Values=WebServer in ASG" --query "Reservations[].Instances[].PublicIpAddress" --output text', returnStdout: true).trim()
                    env.REMOTE_IPS = remoteIPs.split("\\s+")
                    println "IP Addresses: ${env.REMOTE_IPS}" // Вывести адреса для проверки
                }
            }
        }

        stage('Копирование файлов на удаленные сервера') {
            steps {
                script {
                    // Получение списка IP-адресов и копирование файлов на каждый сервер
                    def remoteIPs = sh(script: 'aws ec2 describe-instances --filters "Name=tag:Name,Values=WebServer in ASG" --query "Reservations[].Instances[].PublicIpAddress" --output text', returnStdout: true).trim()
                    remoteIPs.split('\\s+').each { ip ->
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'mkdir /tmp/pet-website'"
                        sh "scp -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem -r ./pet-website/* ubuntu@${ip}:/tmp/pet-website"
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'sudo rsync -a /tmp/pet-website/ /var/www/html/'"
                        sh "ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/my_ssh_key.pem ubuntu@${ip} 'rm -rf /tmp/pet-website'"
                        }
                }
            }
        }
    }

    post {
        always {
            sh 'pwd'
            // Остановка и удаление контейнера после завершения пайплайна
            sh 'docker stop nginx-container || true'
            sh 'docker rm nginx-container || true'
            deleteDir()
        }
    }
}