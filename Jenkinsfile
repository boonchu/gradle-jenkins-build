// Uses Declarative syntax to run commands inside a container.
pipeline {
    agent {
        kubernetes {
            defaultContainer 'gradle'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: gradle
    image: gradle:5.4.1-jdk8-alpine
    command:
    - sleep
    args:
    - 9999
  - name: docker
    image:  docker:dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
      - name: DOCKER_DRIVER
        value: "overlay2"
'''
        }
    }
    stages {
        stage('Clone sources') {
            steps {
                git url: 'https://github.com/boonchu/gradle-jenkins-build.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-server') {
                    sh """
                        echo ${env.SONAR_HOST_URL}
                        cd src && ./gradlew sonarqube
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Docker Build') {
            steps {
                container("docker") {
                   sh """
                      apk update
                      apk add --no-cache py-pip python3-dev libffi-dev openssl-dev gcc libc-dev make cryptography==2.8
                      pip install docker-compose
                      docker info
                      docker version
                      docker-compose build
                   """
                }
            }
        }

    }
}
