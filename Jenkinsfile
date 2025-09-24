pipeline {
  agent {
    docker {
      image 'hdxt25/agent:v1'  
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Build and Test') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        // SONAR_URL = "http://<public-ip>:9000
        SONAR_URL = "http://host.docker.internal:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
  /*  stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "hdxt25/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh "docker build -t ${DOCKER_IMAGE} ."
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }*/
    stage('Build and Push Docker Image') {
    environment {
        DOCKER_IMAGE = "hdxt25/ultimate-cicd:${BUILD_NUMBER}"
    }
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'docker-cred', 
                                             usernameVariable: 'DOCKER_USER', 
                                             passwordVariable: 'DOCKER_PASS')]) {
                sh """
                    # Login to Docker Hub
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    # Build the Docker image
                    docker build -t ${DOCKER_IMAGE} .

                    # Push the Docker image
                    docker push ${DOCKER_IMAGE}

                    # Logout for security
                    docker logout
                """
            }
        }
    }
}
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "spring-boot-app"
            GIT_USER_NAME = "hdxt25"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "hdxt25@gmail.com"
                    git config user.name "himanshu"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
