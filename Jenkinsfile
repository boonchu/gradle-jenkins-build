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
  - name: docker-compose
    image: docker/compose:alpine-1.29.2
    command:
    - sleep
    args:
    - 9999
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
                container("docker-compose") {
                   sh """
                      docker-compose build
                   """
                }
            }
        }

    }
}
