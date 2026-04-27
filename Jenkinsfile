pipeline {
    agent any

    environment {
        IMAGE_NAME = 'mi-app:latest'
        APP_CONTAINER = 'mi-app'
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONAR_PROJECT_KEY = 'mi-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.host.url=${SONAR_HOST_URL}'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Security Hotspot Gate (SonarQube)') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        HOTSPOTS=$(curl -sS -u "${SONAR_AUTH_TOKEN}:" \
                          "${SONAR_HOST_URL}/api/hotspots/search?projectKey=${SONAR_PROJECT_KEY}&status=TO_REVIEW")
                        HOTSPOTS_COUNT=$(echo "$HOTSPOTS" | python3 -c "import json,sys; print(json.loads(sys.stdin.read()).get('paging',{}).get('total',0))")
                        echo "Security Hotspots pendientes: ${HOTSPOTS_COUNT}"
                        if [ "${HOTSPOTS_COUNT}" -gt 0 ]; then
                          echo "Fallo por gate de seguridad: SonarQube detectó Security Hotspots pendientes."
                          exit 1
                        fi
                    '''
                }
            }
        }

        stage('Container Security Scan (Trivy)') {
            steps {
                sh 'trivy image --severity CRITICAL --exit-code 1 --no-progress ${IMAGE_NAME}'
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                sh 'docker rm -f ${APP_CONTAINER} || true'
                sh 'docker run -d --name ${APP_CONTAINER} -p 80:8080 ${IMAGE_NAME}'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline falló: revisar etapas de calidad/seguridad/build.'
        }
        always {
            sh 'docker ps -aq -f name=${APP_CONTAINER} | xargs -r docker rm -f'
            cleanWs()
        }
    }
}
