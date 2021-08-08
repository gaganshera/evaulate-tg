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
                    checkout([$class: 'GitSCM', branches: [[name: '**']], extensions: [], userRemoteConfigs: [[credentialsId: '70c8e005-4a66-4653-b36f-7cb529170c6a', url: 'https://github.com/tarungarg1208/nagp-devops-assign-2']]])
                }
            }
            stage('Build') {
                steps {
                    // One or more steps need to be included within the steps block.
                    bat 'npm install'
                }
            }
            stage('Unit Testing') {
                steps {
                    // One or more steps need to be included within the steps block.
                    bat 'npm test'
                }
            }
            stage('sonar analysis') {
                steps {
                    bat '..\\..\\tools\\hudson.plugins.sonar.SonarRunnerInstallation\\SonarQubeScanner\\bin\\sonar-scanner.bat -Dsonar.host.url=http://localhost:9000 -Dsonar.login=e0dbf4fbb99dbe5b0f0900fade0aac7d1d8299ca'
                }
            }
            stage('Docker image') {
                steps {
                    echo 'Building Docker Image'
                    bat "docker build -t i-${username}-master ."
                }
            }
            stage('Containers') {
                parallel {
                    // One or more stages need to be included within the parallel block.
                    // stage('Pre-Container Check') {
                    // steps {
                    //     bat "docker rm -f run c-tarungarg02-master"
                    // }
                    // },
                    stage('Publish to Docker Hub') {
                    steps {
                        echo 'Tagging and Moving Docker Image'
                        bat "docker tag i-${username}-master ${registry}:${BUILD_NUMBER}"
                        bat "docker tag i-${username}-master ${registry}:master-latest"
                        withDockerRegistry([credentialsId: 'DockerHub', url:'']) {
                            bat "docker push ${registry}:${BUILD_NUMBER}"
                            bat "docker push ${registry}:master-latest"
                        }
                    }
                    }
                }
            }
            stage('Docker deployment') {
                    steps {
                    echo 'Running Docker Image'
                    bat "docker run --name c-${username}-master -d -p=${portmaster}:7100 ${registry}:${BUILD_NUMBER}"
                    }
            }
    }
}
