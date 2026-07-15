pipeline {
    agent any

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = '${GITHUB_USER}/mi-app'
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
        IMAGE_TAG_LATEST = "${REGISTRY}/${IMAGE_NAME}:latest"
        IMAGE_TAG_COMMIT = "${REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}"
        IMAGE_TAG_BUILD = "${REGISTRY}/${IMAGE_NAME}:build-${BUILD_TIMESTAMP}"
    }

    stages {
        stage('Preparación') {
            steps {
                echo '🔧 Preparando entorno...'
                sh 'docker --version'
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Instalación') {
            steps {
                echo '📥 Instalando dependencias...'
                sh 'npm ci'
            }
        }

        stage('Pruebas') {
            steps {
                echo '🧪 Ejecutando pruebas...'
                sh 'npm test'
            }
            post {
                always {
                    // Publicar reportes de tests
                    junit 'test-results/**/*.xml'
                }
            }
        }

        stage('Construcción Docker') {
            steps {
                echo '🐳 Construyendo imagen Docker...'
                script {
                    docker.build("${IMAGE_TAG_COMMIT}")
                }
            }
        }

        stage('Publicación') {
            when {
                branch 'main'
            }
            steps {
                echo '📤 Publicando imagen en GHCR...'
                script {
                    withCredentials([
                        string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')
                    ]) {
                        // Login a GHCR
                        sh """
                            echo \${GITHUB_TOKEN} | docker login ghcr.io -u \${GITHUB_USER} --password-stdin
                        """
                        // Tag y push
                        sh """
                            docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_LATEST}
                            docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_BUILD}
                            docker push ${IMAGE_TAG_COMMIT}
                            docker push ${IMAGE_TAG_LATEST}
                            docker push ${IMAGE_TAG_BUILD}
                        """
                    }
                }
            }
        }

        stage('Verificación') {
            when {
                branch 'main'
            }
            steps {
                echo '✅ Verificando imagen publicada...'
                script {
                    sh """
                        echo "📦 Imagen publicada: ${IMAGE_TAG_COMMIT}"
                        echo "🏷️ Tags disponibles:"
                        echo "  - ${IMAGE_TAG_COMMIT}"
                        echo "  - ${IMAGE_TAG_LATEST}"
                        echo "  - ${IMAGE_TAG_BUILD}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completado exitosamente!'
        }
        failure {
            echo '❌ Pipeline falló!'
        }
        cleanup {
            echo '🧹 Limpiando recursos...'
            sh 'docker image prune -f'
        }
    }
}