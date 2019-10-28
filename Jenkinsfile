def APPS = []
pipeline {
    // If you are running jenkins in a container use "agent { docker { image 'docker:18.09.0-git' }}"
    agent {
        kubernetes {
          label 'docker'
          defaultContainer 'jnlp'
          yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-docker
spec:
  containers:
  - name: docker
    image: docker:18.09.0-git
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    environment {
        GITHUB_HOOK_SECRET = "github-webhook-token-app-mono"
        //DOCKERHUB = credentials('dockerhub-credentials')
        DOCKERHUB_USR = ""
        DOCKERHUB_PSW = ""
    }

    stages {
        stage('configure webook') {
            steps {
                script {
                    setupWebhook()
	        }
	    }
        }

        stage('Find app name to build') {
            steps {
                script {
                    if (REF != "") {
                        VALUESFILE = sh(returnStdout: true, script:'git show --name-only --pretty=""')
                        LIST = VALUESFILE.split('\n')
                        def MAP = [:]
                        for(String file in LIST) {
                            MAP.put(file.split('/')[0], "build");
                        }
                        APPS = MAP.keySet()
                        echo "${APPS}"
                    }
                }
                echo "Changes in:${VALUESFILE}"
                echo "application to build:${APPS}"
            }
        }
        
        stage('Build and push docker image') {
            steps {
                container('docker') {
                    script {
                        sh 'docker login -u $DOCKERHUB_USR -p $DOCKERHUB_PSW'
                        for(String app in APPS) {
                            TO_BUILD = sh (
                                script: "ls ${app}/Dockerfile",
                                returnStatus: true
                            )
                            if (TO_BUILD == 0) {
                                env.APP = app
                                env.GIT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD")
                                sh '''docker build -t infracloud/app-mono-${APP}:${GIT_SHA} ./${APP}/
                                docker push infracloud/app-mono-${APP}:${GIT_SHA}
                                docker rmi infracloud/app-mono-${APP}:${GIT_SHA}
                                '''
                            }
                        }
                        sh 'docker logout'
                    }
                }
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
    }
}

def setupWebhook() {
    properties([
        pipelineTriggers([
            [$class: 'GenericTrigger',
                genericVariables: [
                    [key: 'REF', value: '$.ref'],
                ],
                causeString: 'Triggered on github push',
                token: env.GITHUB_HOOK_SECRET,
                printContributedVariables: true,
                printPostContent: true,
                regexpFilterText: '$REF',
                regexpFilterExpression: 'refs/heads/master'
            ]
        ])
    ])
}
