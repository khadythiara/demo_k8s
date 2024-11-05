pipeline {
    environment {
        imagename = "tambedou/demo-enset-student"
        registryCredential = 'Dockerhub'
        dockerImage = ''
        sonarqubeServerUrl = 'https://4b63-196-207-231-216.ngrok-free.app'
        sonarToken = credentials('sonar-token')
    }
    agent any
    stages {
        stage('Cloning Git') {
            steps {
                git([url: 'https://github.com/Mariamatambedou/docker.git', branch: 'main', credentialsId: 'Github'])
            }
        }
        stage('SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        export PATH=$PATH:/opt/sonar-scanner/bin
                        mvn clean compile
                        sonar-scanner \
                            -Dsonar.projectKey=my_sonar_key \
                            -Dsonar.sources=. \
                            -Dsonar.java.binaries=target \
                            -Dsonar.host.url=${sonarqubeServerUrl} \
                            -Dsonar.login=${sonarToken} \
                            -Dsonar.java.jdkHome=/usr/lib/jvm/java-17-openjdk-amd64  # Spécifiez le chemin de Java 17 si nécessaire
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 30, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build(imagename, ".")
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                sh "docker rmi $imagename:$BUILD_NUMBER"
                sh "docker rmi $imagename:latest"
            }
        }
    }
}
