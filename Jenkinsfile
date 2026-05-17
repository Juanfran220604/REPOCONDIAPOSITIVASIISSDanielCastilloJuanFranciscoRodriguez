pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    parameters {
        string(
            name: 'MD_FILE',
            defaultValue: 'slides/presentacion.md',
            description: 'Ruta del archivo markdown a procesar'
        )
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }
    stages {
        stage('Checkout desde SCM') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                sh '''
                    # Paso vital: Crear la red dentro del motor DinD si no existe
                    docker network create sonarqube_network || true
                    
                    docker run --rm \
                        --network sonarqube_network \
                        -e SONAR_HOST_URL="http://sonarqubep:9000" \
                        -e SONAR_TOKEN=${SONAR_TOKEN} \
                        -v "${WORKSPACE}:/usr/src" \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=marp-slides-project \
                        -Dsonar.sources=/usr/src
                '''
    }
}
        stage('Instalación de dependencias y generación del PDF') {
            environment {
                DOCKER_BUILDKIT = '0'
            }

            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir '.'
                    reuseNode true
                }
            }

            steps {
                sh 'docker pull node:22-bookworm'
            }
        }

        stage('Archivado del artefacto') {
            steps {
                archiveArtifacts artifacts: 'pdf/*.pdf', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }

    post {
        success {
            echo 'PDF generado y archivado correctamente.'
        }
        failure {
            echo 'La ejecución ha fallado.'
        }
        always {
            echo 'Fin del pipeline.'
        }
    }
}