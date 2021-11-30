pipeline{
    agent any
    tools {
      maven 'maven3'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/krishank98/dockeransiblejenkins'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('ansible installation'){
            steps{
                sshagent(['ubuntu-jen']) {
                      sh "echo pwd"
                      sh 'ssh -t -t ec2-user@172.31.43.42 -o StrictHostKeyChecking=no "sudo yum update -y && sudo yum install ansible2 -y"'
                }
            }
        }
        stage('Docker Build'){
            steps{
               // sh "chmod 777 /var/run/docker.sock"
                sh "docker build . -t krish2356/jenkins:${DOCKER_TAG}"
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerhubPwd')]) {
                    sh "docker login -u krish2356 -p ${dockerhubPwd}"
                }
                
                sh "docker push krish2356/jenkins:${DOCKER_TAG} "
            }
        }
        
        stage('Docker-ansible-Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
