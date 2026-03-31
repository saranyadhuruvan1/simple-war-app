pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                sshagent(['github-ssh']) {
                    git branch: 'main', url: 'git@github.com:saranyadhuruvan1/simple-war-app.git'
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }
         stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("simple-app:${BUILD_NUMBER}")
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus_login',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh """
                        mvn deploy \
                        -Dnexus.url=$NEXUS_URL \
                        -Dnexus.user=$NEXUS_USER \
                        -Dnexus.pass=$NEXUS_PASS
                    """
                }
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_URL}
                '''
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                sh '''
                    docker tag ${REPO_NAME}:latest ${ECR_URL}/${REPO_NAME}:${IMAGE_TAG}
                    docker push ${ECR_URL}/${REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "🎉 Build + Nexus Upload completed successfully!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}

