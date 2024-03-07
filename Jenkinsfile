pipeline {
    agent any
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/bpsod10/Shopping-Cart.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
                }
            }
        }

        stage('MVN Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }


        stage('docker image build') {
            environment {
                DOCKER_IMAGE = "bpsod10/shopping-cart:latest"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }

            steps {
                sh "docker build -t ${DOCKER_IMAGE} -f docker/Dockerfile ."
            }
        }




        stage('trivy dockerimage scan') {
            steps {
                sh "trivy image --format table -o trivy-report.html bpsod10/shopping-cart:latest"
            }
        }

        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "bpsod10/shopping-cart:latest"
                REGISTRY_CREDENTIALS = credentials('docker-cred')
            }

            steps {
                script {
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }


    }
}