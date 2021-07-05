// Uses Declarative syntax to run commands inside a container.
pipeline {
    // 'environments'
    // environment {}
    // * end of env *

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
        // https://devopscube.com/declarative-pipeline-parameters/
        stage('Parameters') {
            steps {
                script {
                       properties([
                            parameters([
                                [$class: 'ChoiceParameter', 
                                    choiceType: 'PT_SINGLE_SELECT', 
                                    description: 'Select the branch from the Dropdown List', 
                                    filterLength: 1, 
                                    filterable: false, 
                                    name: 'GIT_BRANCH_NAME', 
                                    script: [
                                        $class: 'GroovyScript', 
                                        fallbackScript: [
                                            classpath: [], 
                                            sandbox: false, 
                                            script: 
                                                "return['Could not get The environemnts']"
                                        ], 
                                        script: [
                                            classpath: [], 
                                            sandbox: false, 
                                            script: 
                                                "return['master','develop','release']"
                                        ]
                                    ]
                                ]
                            ])
                        ])
                } 
            }
        }

        stage('Clone git project repo') {
            steps {
                sh """
					echo "Pulling git branch  + ${GIT_BRANCH_NAME}"
                """
                git branch: "${GIT_BRANCH_NAME}", url: 'https://github.com/boonchu/gradle-jenkins-build.git'
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
