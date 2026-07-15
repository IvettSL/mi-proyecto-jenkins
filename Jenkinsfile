pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'IvettSL/mi-app'
        COMMIT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
        IMAGE_TAG_LATEST = "${REGISTRY}/${IMAGE_NAME}:latest"
        IMAGE_TAG_COMMIT = "${REGISTRY}/${IMAGE_NAME}:${COMMIT_SHA}"
        IMAGE_TAG_BUILD = "${REGISTRY}/${IMAGE_NAME}:build-${BUILD_TIMESTAMP}"
    }

    stages {
        // ============================================
        // STAGE 1: Preparación
        // ============================================
        stage('Preparación') {
            steps {
                echo '🔧 Preparando entorno...'
                sh '''
                    docker --version || echo "Docker no encontrado, usando host.docker.internal"
                    echo "DOCKER_HOST: ${DOCKER_HOST}"
                '''
            }
        }

        // ============================================
        // STAGE 2: Instalación de Node (Usando agente Node)
        // ============================================
        stage('Instalación y Pruebas') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo '📥 Instalando dependencias...'
                sh 'npm ci'
                sh 'npm test'
            }
        }

        // ============================================
        // STAGE 3: Construcción Docker
        // ============================================
        stage('Construcción Docker') {
            steps {
                echo '🐳 Construyendo imagen Docker...'
                script {
                    // Usar Docker directamente con sh en lugar de docker.build
                    sh """
                        docker build -t ${IMAGE_TAG_COMMIT} .
                        docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_LATEST}
                        docker tag ${IMAGE_TAG_COMMIT} ${IMAGE_TAG_BUILD}
                    """
                }
            }
        }

        // ============================================
        // STAGE 4: Publicación
        // ============================================
        stage('Publicación') {
            when {
                branch 'main'
            }
            steps {
                echo '📤 Publicando imagen...'
                script {
                    withCredentials([
                        string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')
                    ]) {
                        sh """
                            echo \${GITHUB_TOKEN} | docker login ghcr.io -u \${GITHUB_USER} --password-stdin
                            docker push ${IMAGE_TAG_COMMIT}
                            docker push ${IMAGE_TAG_LATEST}
                            docker push ${IMAGE_TAG_BUILD}
                        """
                    }
                }
            }
        }

        // ============================================
        // STAGE 5: Verificación
        // ============================================
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
            sh 'docker image prune -f || true'
        }
    }
}