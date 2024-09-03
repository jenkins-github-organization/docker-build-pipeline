@Library('shared-library@main') _
pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.8.4-openjdk-17
                    command:
                    - cat
                    tty: true
                    volumeMounts:
                    - name: maven-cache
                      mountPath: /root/.m2
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                    - name: docker-config
                    mountPath: /kaniko/.docker
                  - name: hadolint
                    image: hadolint/hadolint:latest-debian
                    command:
                    - sleep
                    args:
                    - "3600"
                  - name: trivy
                    image: aquasec/trivy
                    command:
                    - sleep
                    args:
                    - "3600"
                  volumes:
                  - name: maven-cache
                    hostPath:
                        path: /home/jenkins/cache
                  - name: docker-config
                    emptyDir: {}
            '''
        }
    }

    environment {
        DOCKER_HUB_REPO = "test"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/kubernetes-learning-projects/kube-petclinc-app.git"
                )
            }
        }
        stage('Lint Dockerfile') {
            steps {
                container('hadolint') {
                    hadoLint()
                }
            }
        }
        stage('Build with Maven') {
            steps {
                container('maven') {
                    script {
                        maven.build()
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    script {
                        kaniko.build()
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    container('trivy') {
                        script {
                            trivyScan.test()
                        }
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                container('kaniko') {
                    script {
                        kaniko.push('aswinvj/test', "1.0.${BUILD_NUMBER}", 'docker-hub-credentials')
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                trivyNotification('trivy-report.html', 'aswin@crunchops.com')
            }
        }
    }
}
