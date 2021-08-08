pipeline {
    agent any
    options {
            timestamps()
            timeout(60)
    }
    environment {
        registry = 'tarungarg1208/nagp-devops-assign-2'
        username = 'tarungarg02'
        portmaster = 7200
        portdevelop = 7300
        CREDENTIALS_ID = 'nagp-gcp-key'
        LOCATION = 'us-central1-c'
        CLUSTER_NAME = 'nagp-assignment'
        PROJECT_ID = 'nagp-devops-assignment'
    }
    tools {
        nodejs 'nodejs'
        jdk 'Java'
        dockerTool 'Test_Docker'
    }
    stages {
            stage('checkout') {
                steps {
                    // One or more steps need to be included within the steps block.
                    // checkout([$class: 'GitSCM', branches: [[name: '**']], extensions: [], userRemoteConfigs: [[credentialsId: '70c8e005-4a66-4653-b36f-7cb529170c6a', url: 'https://github.com/tarungarg1208/nagp-devops-assign-2']]])
                    checkout scm
                }
            }
            stage('Build') {
                steps {
                    // One or more steps need to be included within the steps block.
                    bat 'npm install'
                }
            }
            stage('Unit Testing') {
                when { branch 'master' }
                steps {
                    // One or more steps need to be included within the steps block.
                    bat 'npm test'
                }
            }
            stage('sonar analysis') {
                when { branch 'develop' }
                steps {
                    bat '..\\..\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\SonarQubeScanner\\bin\\sonar-scanner.bat -Dsonar.host.url=http://localhost:9000 -Dsonar.login=658cd6afba259bd114439d623d10e01af79523cc'
                }
            }
            stage('Docker image') {
                steps {
                    echo 'Building Docker Image'
                    bat "docker build -t i-${username}-${env.BRANCH_NAME} ."
                }
            }
            stage('Containers') {
            steps {
                parallel(
                    'Publish to Docker Hub': {
                        echo 'Tagging and Moving Docker Image'
                        bat "docker tag i-${username}-${env.BRANCH_NAME} ${registry}:${env.BRANCH_NAME}-${BUILD_NUMBER}"
                        bat "docker tag i-${username}-${env.BRANCH_NAME} ${registry}:${env.BRANCH_NAME}-latest"
                        withDockerRegistry([credentialsId: 'DockerHub', url:'']) {
                            bat "docker push ${registry}:${env.BRANCH_NAME}-${BUILD_NUMBER}"
                            bat "docker push ${registry}:${env.BRANCH_NAME}-latest"
                        }
                    },
                    'Pre-container check': {
                        script {
                        try {
                            bat "docker rm -f c-${username}-${env.BRANCH_NAME}"
                            } 
                        catch (Exception e) {
                            echo 'No Container to remove'
                        }
                        }
                    }
                )
            }
            }
            stage('Docker deployment') {
                    steps {
                    echo 'Running Docker Image'
                    script {
                    if (env.BRANCH_NAME == 'master') {
                        bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${portmaster}:7100 ${registry}:${env.BRANCH_NAME}-${BUILD_NUMBER}"
                    }
                    else {
                        bat "docker run --name c-${username}-${env.BRANCH_NAME} -d -p=${portdevelop}:7100 ${registry}:${env.BRANCH_NAME}-${BUILD_NUMBER}"
                    }
                    }
                    }
            }
            stage('Kubernetes Deployment') {
                    steps {
                    echo 'Deploying to Kubernetes'
                    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'k8s/deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: false])
                    }
            }
    }
}
