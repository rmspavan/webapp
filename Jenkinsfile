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
                git branch: 'staging', credentialsId: 'github', url: 'https://github.com/rmspavan/webapp.git'
            }
        }
    stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
   stage('Docker Build'){
            steps{
                sh "docker build . -t rmspavan/app:${DOCKER_TAG} "
            }
        }
    stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')])
                {
                    sh "docker login -u rmspavan -p ${dockerHubPwd}"
                }
                
                sh "docker push rmspavan/app:${DOCKER_TAG} "
            }
        }
    stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'Staging-Server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }    
    }
    
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}

