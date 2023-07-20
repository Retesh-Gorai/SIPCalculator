pipeline {
  agent {
    docker {
      //using below image to run docker pipeline plugin
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Retesh-Gorai/SIPCalculator.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // building the project and creating a JAR file: not using mvn clean install, as we won't push the artifact to nexus/ artifactory
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://35.154.201.175:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube-cred', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "retesh/sip-calculator:${BUILD_NUMBER}"  
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "SIPCalculator"
            GIT_USER_NAME = "Retesh-Gorai"
        }
        steps {
            withCredentials([string(credentialsId: 'github-cred', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "retesh.retesh.gorai@gmail.com"
                    git config user.name "Retesh Gorai"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifests/deployment.yml
                    git add manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
