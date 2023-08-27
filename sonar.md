--- Docker Installation ---

sudo apt update
sudo apt install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
  sudo apt update
  
  sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
  
  sudo apt install docker-compose -y
  
service docker restart

sudo usermod -aG docker $USER

newgrp docker

sudo chmod 666 /var/run/docker.sock

sudo systemctl restart docker

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

docker ps

Access sonar IP:9000

user/pass   admin/admin
========================================================

pipeline {
    agent any
    
    tools {
        
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git-Checkout') {
            steps {
                git 'https://github.com/jaiswaladi2468/BoardgameListingWebApp.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Package') {
            steps {
                sh "mvn package"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                
                withSonarQubeEnv('sonar') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Aditya \
                            -Dsonar.projectKey=Aditya  -Dsonar.java.binaries=. '''
                }
               
            }
        }
        
        
    }
}



