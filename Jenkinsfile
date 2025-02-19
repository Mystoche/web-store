pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-token')  // Assurez-vous d'ajouter le token dans les credentials Jenkins
        DOCKER_IMAGE_NAME = 'dulcinee/web-store'  // L'image Docker à scanner
        DOCKERFILE_PATH = './web-page/Dockerfile'  // Le chemin vers votre Dockerfile
    }

    stages {
        stage('Cloner le code') {
            steps {
                git 'https://github.com/Mystoche/web-store.git'
            }
        }

        stage('Construire l\'image Docker') {
            steps {
                script {
                    // Construire l'image Docker à partir du Dockerfile
                    echo 'Building Docker image...'
                    sh '''
                        docker build -t $DOCKER_IMAGE_NAME -f $DOCKERFILE_PATH .
                    '''
                }
            }
        }

        stage('Scanner l\'image Docker avec Trivy') {
            steps {
                script {
                    // Scanner l'image Docker pour détecter les vulnérabilités avec Trivy
                    echo 'Scanning Docker image with Trivy...'
                    sh '''
                        trivy image --exit-code 1 --severity HIGH,CRITICAL --no-progress $DOCKER_IMAGE_NAME
                    '''
                }
            }
        }

        stage('Analyse de sécurité avec SonarQube') {
            steps {
                script {
                    // Exécution de l'analyse de sécurité avec SonarQube
                    echo 'Running SonarQube analysis...'
                    withSonarQubeEnv('Sonarqube-server') {  // Assurez-vous que le nom du serveur SonarQube correspond à la configuration Jenkins
                        sh '''
                            mvn clean install sonar:sonar -Dsonar.projectKey=web-shop -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Vérification du Quality Gate de SonarQube') {
            steps {
                script {
                    // Attendre la fin de l'analyse SonarQube et vérifier le Quality Gate
                    echo 'Waiting for SonarQube Quality Gate status...'
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Notifier le résultat') {
            steps {
                script {
                    // Vérification de l'état du build et notification en cas d'échec
                    if (currentBuild.result == 'FAILURE') {
                        error "Des vulnérabilités critiques ou un échec de qualité ont été détectés."
                    } else {
                        echo 'Le code et l\'image Docker sont sécurisés et passent les tests de qualité.'
                    }
                }
            }
        }
    }

    post {
        always {
            // Actions à réaliser après la fin du pipeline, par exemple, nettoyage
            cleanWs()  // Nettoyer l'espace de travail à la fin
        }
    }
}
