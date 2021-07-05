// Uses Declarative syntax to run commands inside a container.
pipeline {
    // 'environments'
    // environment {}
    // * end of env *

    // https://devopscube.com/declarative-pipeline-parameters/
    parameters {
        string(name: 'GIT_BRANCH_NAME', defaultValue: 'master', description: 'Git branch to use for the build.')
    }
    // * end of params *

    // 'agent' statement
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
    image:  boonchu/docker:dind
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
    // * end of agent *

    // stage declarative statements
    stages {
        stage('Clone git project repo') {
            steps {
                sh """
					echo "Pulling git branch  + ${params.GIT_BRANCH_NAME}"
                """
                git branch: "${params.GIT_BRANCH_NAME}", url: 'https://github.com/boonchu/gradle-jenkins-build.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-server') {
                    sh """
                        echo ${env.SONAR_HOST_URL}
                        cd src/gradle-java-build && ./gradlew sonarqube
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
                      docker-compose build
                   """
                }
            }
        }
    }
    // * end of stage *
}
