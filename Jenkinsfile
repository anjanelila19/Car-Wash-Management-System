
def img
pipeline {
    environment {
        registry = "lilaramdocker/myprojectdemo" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-login'
        dockerImage = ''
        container_name='my_app'
    }
    agent any
    stages {
        stage('checkout') {
            steps {
                git 'https://github.com/anjanelila19/Car-Wash-Management-System.git'
            }
        }

        stage ('Stop previous running container'){
            steps{
                 sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${container_name} | awk \'{print $1}\')'
                 sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                 sh returnStatus: true, script: 'docker rm ${container_name}'
                  sh 'ls'
                  
            }
        }


        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }



        stage('Test - Run Docker Container on Jenkins node') {
           steps {

                sh label: '', script: "docker run -d --name ${container_name} -p 5000:5000 ${img}"
          }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Test Server') {
            steps {
                script {
                    def stopcontainer = "docker stop ${container_name}"
                    def delcontName = "docker rm ${container_name}"
                    def delimages = 'docker image prune -a --force'
                    def drun = "docker run -d --name ${container_name} -p 5000:5000 ${img}"
                    println "${drun}"
                    sshagent(['target_server']) {
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no docker@3.76.216.225 ${stopcontainer} "
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no docker@3.76.216.225 ${delcontName}"
                        sh returnStatus: true, script: "ssh -o StrictHostKeyChecking=no docker@3.76.216.225 ${delimages}"

                    // some block
                        sh "ssh -o StrictHostKeyChecking=no docker@3.76.216.225 ${drun}"
                    }
                }
            }
        }


    }
}  
