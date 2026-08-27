pipeline {
    agent any

    parameters {
        string(name: 'REGISTRY_URL', defaultValue: 'docker.io/seuusuario', description: 'Registro de imagens (Docker Hub, GHCR, etc.)')
        choice(name: 'DEPLOY', choices: ['sim', 'nao'], description: 'Publicar (deploy) após build + testes?')
    }

    environment {
        IMAGE_NAME   = 'jenkins-test-name'
        IMAGE_TAG    = "${params.REGISTRY_URL}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
        CONTAINER    = 'jenkins-test'
        PORT_HOST    = '80'
        PORT_CONT    = '80'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '4'))
        timeout(time: 15, unit: 'MINUTES')
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                echo ' Buscando código fonte do repositório'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Empacotando o site estático com Docker'
                sh 'ls -la'
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Test') {
            steps {
                echo 'Validando a imagem gerada'
                sh "docker run --rm --name ${CONTAINER}-check ${IMAGE_NAME}:latest ls /usr/share/nginx/html/index.html"
            }
        }

        stage('Package') {
            steps {
                echo ' Marcando imagem com tag de versão e subindo para o registro'
                sh "docker tag ${IMAGE_NAME}:latest ${IMAGE_TAG}"
                withCredentials([usernamePassword(credentialsId: 'docker-hub',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy') {
            when { expression { params.DEPLOY == 'sim' } }
            steps {
                echo '} Publicando container na porta 80 {'
                sh "docker rm -f ${CONTAINER} || true"
                sh "docker run -d --name ${CONTAINER} -p ${PORT_HOST}:${PORT_CONT} ${IMAGE_TAG}"
            }
        }
    }

    post {
        success {
            echo 'Pipeline concluído com SUCESSO'
        }
        failure {
            echo 'Pipeline FALHOU verifique os logs'
        }
        always {
            cleanWs()
        }
    }
}