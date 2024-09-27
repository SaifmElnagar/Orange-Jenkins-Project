pipeline {
    agent {
        kubernetes {
            label 'my-agent'
            defaultContainer 'jnlp'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              name: jenkins-agent
            spec:
              serviceAccountName: jenkins-admin
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
              - name: docker
                image: docker:20.10.7
                command:
                - cat
                tty: true
                volumeMounts:
                - name: docker-socket
                  mountPath: /var/run/docker.sock
              - name: kubectl
                image: kumargaurav522/jnlp-kubectl-slave:latest
                command:
                - cat
                tty: true
              - name: git
                image: bitnami/git:latest
                command:
                - cat
                tty: true
              volumes:
              - name: docker-socket
                hostPath:
                  path: /var/run/docker.sock
            """
        }
    }
    environment {
        GIT_REPO1 = 'https://github.com/halimo22/backend'
        GIT_REPO2 = 'https://github.com/halimo22/proxy'
        GIT_REPO3 = 'https://github.com/halimo22/Jenkins_project'
        DOCKER_REGISTRY = 'halimo2'
    }
    stages {
        stage('Pull and Build First Image') {
            steps {
                container('git') {  // Use the 'git' container to clone the repo
                    script {
                        sh "git clone ${GIT_REPO1} repo1"
                    }
                }
                container('docker') {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            // Login to Docker Registry
                            sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"

                            // Build and push the Docker image from the first repo
                            dir('repo1') {
                                sh 'docker build -t backend .'
                                sh "docker tag backend ${DOCKER_REGISTRY}/backend"
                                sh "docker push ${DOCKER_REGISTRY}/backend"
                            }
                        }
                    }
                }
            }
        }
        stage('Pull and Build Second Image') {
            steps {
                container('git') {
                    script {
                        sh "git clone ${GIT_REPO2} repo2"
                    }
                }
                container('docker') {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            // Login to Docker Registry
                            sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                            
                            // Build and push the Docker image from the second repo
                            dir('repo2') {
                                sh 'docker build -t proxy .'
                                sh "docker tag proxy ${DOCKER_REGISTRY}/proxy"
                                sh "docker push ${DOCKER_REGISTRY}/proxy"
                            }
                        }
                    }
                }
            }
        }
        stage('Pull Configuration and Apply to Kubernetes') {
            steps {
                container('git') {
                    script {
                        sh "git clone ${GIT_REPO3} repo3"
                    }
                }
                container('kubectl') {
                    script {
                        // Apply the Kubernetes configuration
             sh 'kubectl apply -f repo3/K8S'
               
            }
                    
                }
            }
     
        }
    }
}
