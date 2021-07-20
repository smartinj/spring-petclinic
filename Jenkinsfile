pipeline {
    agent any
    tools {
        jdk 'JDK 1.8'
    }
    environment {
        SPRING_PROFILES_ACTIVE = "ci"
    }
    stages {
        stage('Initialize') {
            steps {
                sh "printenv"
            }
        }
        stage('Package') {
            when {
                expression {
                    currentBuild.resultIsBetterOrEqualTo('SUCCESS')
                }
            }
            steps {
                withMaven(maven: 'M3', options: [jacocoPublisher(disabled: true)]) {
                    sh "mvn -DskipTests package"
                }
            }
        }
        stage('Deploy [Docker]') {
            when {
                expression {
                    currentBuild.resultIsBetterOrEqualTo('SUCCESS')
                }
            }
            stages {
                stage('Build image') {
                    steps {
                        dir(".") {
                            withMaven(maven: 'M3', options: [jacocoPublisher(disabled: true)]) {
                                sh "mvn dockerfile:build -Ddockerfile.skip=false"
                            }
                        }
                    }
                }
                stage('Deploy [docker-registry:5000]') {
                    steps {
                        script {
                            docker.withTool('19.03.9') {
                                docker.withRegistry('https://docker-registry:5000', 'registry-id') {
                                    def image = docker.image('docker-registry:5000/petclinic:v1')
                                    image.push "v1"
                                }
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy [k8s]') {
            when {
                expression {
                    currentBuild.resultIsBetterOrEqualTo('SUCCESS')
                }
            }
            stages {
                stage('kubectl') {
                    steps {
                        sh "kubectl apply -f file.yaml"
                    }
                }
            }
        }
    }
}