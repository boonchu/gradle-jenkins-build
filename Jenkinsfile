// Uses Declarative syntax to run commands inside a container.
pipeline {
    // https://devopscube.com/declarative-pipeline-parameters/
    parameters {
        string(name: 'GIT_BRANCH_NAME', defaultValue: 'master', description: 'Git branch to use for the build.')
        string(name: 'IMAGE_TAG', defaultValue: '1.0.0-pre', description: 'Image tag to use for the build.')
    }
    // * end of params *

    // 'environments'
    environment {
		APP_NAME="api-db"
        TAG= "${params.IMAGE_TAG}"
        REGISTRY_PATH="docker-registry:5000/images/example"
        IMAGE_FULL_PATH= "$REGISTRY_PATH/$APP_NAME:$TAG"
	}
    // * end of env *

    // 'agent' statement
    /*
     * agent {
     *     docker {
     *         label 'gradle-docker-slave'
     *         image 'docker-registry:5000/images/example/gradle-builder:jdk11'
     *         args '-u root \
     *               -v /var/run/docker.sock:/var/run/docker.sock \
     *               -v "$PWD":/usr/app'
     *         reuseNode true
     *     }
	 * }
     */

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
        
        stage ('Test & Build Artifact') {
            steps {
                sh """
					cd src/gradle-java-build && ./gradlew --build-file "build.gradle" clean build -x test'
				"""
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
