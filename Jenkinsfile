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
            '''
        }
    }

    environment {
        DOCKER_HUB_REPO = "test"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage('Git Checkout') {
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/spring-projects/spring-petclinic.git"
                )
            }
        }
        stage('test') {
            steps {
                sh "sleep 3600"
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
                container('trivy') {
                    script {
                        trivyScan.test()
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def reportPath = "trivy-report.html"
                def recipient = "aswin@crunchops.com"
                trivyNotification(reportPath, recipient)
            }
        }
    }
}
