pipeline {
    agent any

    environment {
        DOCKERHUB_REPO_FRONTEND = 'sagarsmanjunath/jerney-frontend'
        DOCKERHUB_REPO_BACKEND  = 'sagarsmanjunath/jerney-backend'

        IMAGE_TAG = "${BUILD_NUMBER}"

        SCANNER_HOME = tool 'sonar-scanner'

        K8S_NAMESPACE = 'jerney'
        K8S_MANIFEST = 'k8s/manifest.yaml'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'devops',
                    credentialsId: 'git-cred',
                    url: 'https://github.com/sagar-smanjunath/Jerney.git'
            }
        }

        stage('Gitleaks Secret Scan') {
            steps {
                sh '''
                    gitleaks detect \
                      --source . \
                      --report-format sarif \
                      --report-path gitleaks-report.sarif \
                      --redact
                '''
            }
        }

        stage('Frontend npm Install') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Frontend Tests') {
            steps {
                dir('frontend') {
                    sh 'npm test -- --run'
                }
            }
        }

        stage('Backend npm Install') {
            steps {
                dir('backend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Backend Tests') {
            steps {
                dir('backend') {
                    sh 'npm test'
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                    trivy fs \
                      --scanners vuln,secret,misconfig \
                      --format table \
                      -o trivy-fs-report.txt \
                      .
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectName=Jerney \
                          -Dsonar.projectKey=Jerney \
                          -Dsonar.sources=frontend/src,backend/src \
                          -Dsonar.exclusions=**/node_modules/**,**/dist/**,**/coverage/**
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate(
                        abortPipeline: true,
                        credentialsId: 'sonar-token'
                    )
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKERHUB_REPO_FRONTEND}:${IMAGE_TAG} \
                      -t ${DOCKERHUB_REPO_FRONTEND}:latest \
                      ./frontend
                '''
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKERHUB_REPO_BACKEND}:${IMAGE_TAG} \
                      -t ${DOCKERHUB_REPO_BACKEND}:latest \
                      ./backend
                '''
            }
        }

        stage('Trivy Frontend Image Scan') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --format table \
                      -o trivy-frontend-image-report.txt \
                      ${DOCKERHUB_REPO_FRONTEND}:${IMAGE_TAG}
                '''
            }
        }

        stage('Trivy Backend Image Scan') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --format table \
                      -o trivy-backend-image-report.txt \
                      ${DOCKERHUB_REPO_BACKEND}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Images To Docker Hub') {
            steps {
                withDockerRegistry(
                    credentialsId: 'docker-cred',
                    toolName: 'docker'
                ) {
                    sh '''
                        docker push ${DOCKERHUB_REPO_FRONTEND}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO_FRONTEND}:latest

                        docker push ${DOCKERHUB_REPO_BACKEND}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO_BACKEND}:latest
                    '''
                }
            }
        }

        stage('Kubeaudit Scan') {
            steps {
                sh '''
                    kubeaudit all \
                      -f ${K8S_MANIFEST} \
                      > kubeaudit-report.txt
                '''
            }
        }

        stage('Update Kubernetes Image Tags') {
            steps {
                sh '''
                    sed -i \
                      "s|sagarsmanjunath/jerney-frontend:latest|sagarsmanjunath/jerney-frontend:${IMAGE_TAG}|g" \
                      ${K8S_MANIFEST}

                    sed -i \
                      "s|sagarsmanjunath/jerney-backend:latest|sagarsmanjunath/jerney-backend:${IMAGE_TAG}|g" \
                      ${K8S_MANIFEST}

                    echo "Kubernetes images:"
                    grep "image:" ${K8S_MANIFEST}
                '''
            }
        }

        stage('Kubernetes Manifest Validation') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        kubectl apply \
                          --dry-run=server \
                          -f ${K8S_MANIFEST}
                    '''
                }
            }
        }

        stage('Deploy To EKS') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        kubectl apply -f ${K8S_MANIFEST}
                    '''
                }
            }
        }

        stage('Backend Rollout Verification') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        kubectl rollout status \
                          deployment/jerney-backend \
                          -n ${K8S_NAMESPACE} \
                          --timeout=5m
                    '''
                }
            }
        }

        stage('Frontend Rollout Verification') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        kubectl rollout status \
                          deployment/jerney-frontend \
                          -n ${K8S_NAMESPACE} \
                          --timeout=5m
                    '''
                }
            }
        }

        stage('Kubernetes Health Checks') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        echo "========== PODS =========="
                        kubectl get pods -n ${K8S_NAMESPACE} -o wide

                        echo "========== SERVICES =========="
                        kubectl get svc -n ${K8S_NAMESPACE}

                        echo "========== DEPLOYMENTS =========="
                        kubectl get deployments -n ${K8S_NAMESPACE}

                        echo "========== HPA =========="
                        kubectl get hpa -n ${K8S_NAMESPACE}

                        echo "========== PVC =========="
                        kubectl get pvc -n ${K8S_NAMESPACE}

                        echo "========== INGRESS =========="
                        kubectl get ingress -n ${K8S_NAMESPACE}
                    '''
                }
            }
        }

        stage('Verify HPA') {
            steps {
                withKubeConfig(
                    credentialsId: 'k8-cred',
                    namespace: "${K8S_NAMESPACE}"
                ) {
                    sh '''
                        echo "========== BACKEND HPA =========="
                        kubectl describe hpa jerney-backend-hpa \
                          -n ${K8S_NAMESPACE}

                        echo "========== FRONTEND HPA =========="
                        kubectl describe hpa jerney-frontend-hpa \
                          -n ${K8S_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        always {
            script {

                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'

                def bannerColor =
                    pipelineStatus.toUpperCase() == 'SUCCESS'
                    ? 'green'
                    : 'red'

                def body = """
                    <html>
                    <body>

                    <div style="border: 4px solid ${bannerColor};
                                padding: 10px;">

                        <h2>
                            ${jobName} - Build ${buildNumber}
                        </h2>

                        <div style="background-color: ${bannerColor};
                                    padding: 10px;">

                            <h3 style="color: white;">
                                Pipeline Status:
                                ${pipelineStatus.toUpperCase()}
                            </h3>

                        </div>

                        <p>
                            <b>Application:</b>
                            Jerney Blog Application
                        </p>

                        <p>
                            <b>Branch:</b>
                            devops
                        </p>

                        <p>
                            <b>Frontend Image:</b>
                            ${DOCKERHUB_REPO_FRONTEND}:${IMAGE_TAG}
                        </p>

                        <p>
                            <b>Backend Image:</b>
                            ${DOCKERHUB_REPO_BACKEND}:${IMAGE_TAG}
                        </p>

                        <p>
                            Check the
                            <a href="${BUILD_URL}">
                                Jenkins Console Output
                            </a>
                        </p>

                    </div>

                    </body>
                    </html>
                """

                emailext(
                    subject:
                        "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",

                    body: body,

                    to: 'appusagar077@gmail.com',

                    mimeType: 'text/html',

                    attachmentsPattern:
                        'gitleaks-report.sarif,' +
                        'trivy-fs-report.txt,' +
                        'trivy-frontend-image-report.txt,' +
                        'trivy-backend-image-report.txt,' +
                        'kubeaudit-report.txt'
                )
            }
        }
    }
}
