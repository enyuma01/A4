pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static Code Analysis') {
            steps {
                script {
                    sh "mvn sonar:sonar -Dsonar.host.url=${SONARQUBE_URL}"
                }
            }
        }
        stage('Dependency Check') {
            steps {
                script {
                    sh "dependency-check --project my-app --scan ."
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t my-app .'
                    sh "docker tag my-app ${DOCKER_REGISTRY}/my-app:latest"
                    sh "docker push ${DOCKER_REGISTRY}/my-app:latest"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
        stage('Dynamic Application Security Testing') {
            steps {
                script {
                    sh "zap-cli quick-scan --self-contained --start-options '-config api.disablekey=true' ${GITHUB_REPO}"
                }
            }
        }
        stage('Monitor') {
            steps {
                script {
                    sh "curl -X POST ${ELK_URL}/log -d @log.json"
                }
            }
        }
        stage('Infrastructure as Code') {
            steps {
                script {
                    dir(TERRAFORM_DIR) {
                        sh 'terraform init'
                        sh 'terraform apply -auto-approve'
                    }
                }
            }
        }
        stage('Vulnerability Management') {
            steps {
                script {
                    sh 'nessus scan launch my-scan'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
