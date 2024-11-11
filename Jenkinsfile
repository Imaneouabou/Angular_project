pipeline {
    agent any

    environment {
        STATIC_ADDRESS = 'http://example.com' // Adresse de déploiement
        SKIP_SAST = 'false' // Exécute le SAST
        SKIP_CONTAINER_SCAN = 'false' // Exécute le scan de conteneur
        SKIP_SCA = 'false' // Exécute le SCA
        DOCKER_IMAGE = 'angular_pipeline'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Compile') {
            agent {
                docker { image 'node:20.17.0-alpine' }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
                archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true // Partage les artefacts de build
                stash includes: 'node_modules/**', name: 'node-modules' // Partage des dépendances installées
            }
        }

        stage('Unit Test') {
            agent {
                docker { image 'node:20.17.0-alpine' }
            }
            steps {
                unstash 'node-modules'
                sh 'npm run test'
            }
        }

        stage('Docker Build and Push') {
            agent {
                docker { image 'docker:latest' }
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub') {
                        def customImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                        customImage.push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline exécuté avec succès !'
        }
        failure {
            echo 'Échec dans une étape du pipeline.'
        }
    }
}
