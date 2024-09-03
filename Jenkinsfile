@Library('shared-library@main') _
pipeline {
    agent {
        label 'buildtools'
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
                    container('trivy') {
                        script {
                            trivyScan.kaniko()
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
