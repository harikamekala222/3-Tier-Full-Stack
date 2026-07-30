pipeline {
    agent any

    tools {
        nodejs 'node22'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'harikamekala222/camp:latest'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-account',
                    url: 'https://github.com/harikamekala222/3-Tier-Full-Stack.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Campground \
                        -Dsonar.projectName=Campground
                    """
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o image-report.html $IMAGE_NAME'
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker compose down || true
                    docker compose up -d
                '''
            }
        }
    }

    post {

        always {
            archiveArtifacts artifacts: '*.html', allowEmptyArchive: true
        }

        success {
            echo '========================================='
            echo 'Pipeline Executed Successfully'
            echo '========================================='
        }

        failure {
            echo '========================================='
            echo 'Pipeline Failed'
            echo '========================================='
        }
    }
}
