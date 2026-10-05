pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_TAG = "pendiente"
        NEXUS_REGISTRY = "localhost:9080"
        NEXUS_MAVEN_REPO = "http://nexus:8081/repository/maven-releases/"
        NEXUS_CREDENTIALS_ID = "nexus-credentials"
    }

    stages {
        stage('Checkout & Test') {
            steps {
                checkout scm
                script {
                    def shortCommit = sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${shortCommit}"
                }
                dir('codigo_base/backend') {
                    sh 'mvn -B test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'codigo_base/backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package & Tag Inmutable') {
            steps {
                dir('codigo_base/backend') {
                    sh 'mvn -B versions:set -DnewVersion=${IMAGE_TAG} -DgenerateBackupPoms=false'
                    sh 'mvn -B package -DskipTests'
                    sh 'docker build -t ${NEXUS_REGISTRY}/studytrack-api:${IMAGE_TAG} .'
                }
                dir('codigo_base/frontend') {
                    sh 'docker build -t ${NEXUS_REGISTRY}/studytrack-frontend:${IMAGE_TAG} .'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: env.NEXUS_CREDENTIALS_ID, usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        printf '<settings><servers><server><id>nexus</id><username>%s</username><password>%s</password></server></servers></settings>' "$NEXUS_USER" "$NEXUS_PASS" > "$WORKSPACE/settings-ci.xml"
                        (cd codigo_base/backend && mvn -B -s "$WORKSPACE/settings-ci.xml" deploy -DskipTests)
                        rm -f "$WORKSPACE/settings-ci.xml"
                        echo "$NEXUS_PASS" | docker login ${NEXUS_REGISTRY} -u "$NEXUS_USER" --password-stdin
                        docker push ${NEXUS_REGISTRY}/studytrack-api:${IMAGE_TAG}
                        docker push ${NEXUS_REGISTRY}/studytrack-frontend:${IMAGE_TAG}
                        docker logout ${NEXUS_REGISTRY}
                    '''
                }
            }
        }

        stage('Deploy & Smoke Test') {
            steps {
                sh '''
                    export NEXUS_REGISTRY IMAGE_TAG
                    docker compose -f codigo_base/deploy/docker-compose.yml down || true
                    docker compose -f codigo_base/deploy/docker-compose.yml up -d
                    docker network connect product-network jenkins || true
                    for i in $(seq 1 20); do
                        if curl -fs http://studytrack-backend:8080/api/tasks > /dev/null; then
                            echo "Smoke test OK"
                            exit 0
                        fi
                        echo "Intento $i fallido"
                        sleep 5
                    done
                    docker compose -f codigo_base/deploy/docker-compose.yml logs
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline finalizado en verde. Artefactos publicados con tag: ${env.IMAGE_TAG}"
        }
        failure {
            echo "El pipeline falló. Revise los logs de la etapa correspondiente antes de reintentar."
        }
    }
}
