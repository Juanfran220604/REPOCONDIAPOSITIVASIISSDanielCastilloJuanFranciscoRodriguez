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
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        docker network create sonarqube_network || true
                        
                        docker run --rm \
                            --network sonarqube_network \
                            -e SONAR_HOST_URL="http://sonarqubep:9000" \
                            -e SONAR_TOKEN=${SONAR_TOKEN} \
                            -v "${WORKSPACE}:/usr/src" \
                            sonarsource/sonar-scanner-cli \
                            -Dsonar.projectKey=marp-slides-project \
                            -Dsonar.sources=. \
                            -Dsonar.inclusions="**/*" \
                            -Dsonar.scm.disabled=true
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
    }
}
        stage('Instalación de dependencias y generación del PDF') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir '.'
                    reuseNode true
                }
            }

            steps {
                sh '''
                node --version
                npm --version

                if [ ! -f "${MD_FILE}" ]; then
                    echo "ERROR: no existe el archivo ${MD_FILE}"
                    exit 1
                fi

                export HOME="$WORKSPACE"
                export npm_config_cache="$WORKSPACE/.npm"
                mkdir -p "$npm_config_cache"
                mkdir -p pdf

                npx @marp-team/marp-cli ${MD_FILE} --pdf --allow-local-files -o pdf/output.pdf

                ls -lh pdf
                '''
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