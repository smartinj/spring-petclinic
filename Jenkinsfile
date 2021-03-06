pipeline {
    agent any
    tools {
        jdk 'JDK 1.8'
    }
    environment {
        IMAGE_TAG = "000"
    }    
    stages {
        stage('Initialize') {
            steps {
                script{
                    IMAGE_TAG = Math.abs(new Random().nextInt() % 900) +100
                }
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
                    sh "mvn clean -Dcheckstyle.skip -DskipTests package"
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
                        sh "printenv"
                        script {
                            docker.withTool('19.03.9') {
                                docker.withRegistry('https://docker-registry:5000', 'registry-id') {
                                    def image = docker.image('docker-registry:5000/petclinic:latest')
                                    image.push "${IMAGE_TAG}"
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
                        withKubeConfig([credentialsId: 'kube-config', serverUrl: 'https://10.10.10.250:6443']) {
                              sh ".tools/ytt -v image=docker-registry:5000/petclinic:${IMAGE_TAG} -f .tools/overlay-image.yaml -f k8s/petclinic-deployment.yaml | kubectl apply -f -"
                        }
                    }
                }
            }
        }
    }
}