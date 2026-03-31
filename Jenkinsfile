pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_URL = "160180440306.dkr.ecr.us-east-1.amazonaws.com"
        REPO_NAME = "simple-war-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                sshagent(['github-ssh']) {
                    git branch: 'main', url: 'git@github.com:saranyadhuruvan1/simple-war-app.git'
                }
            }
        }

        stage('Build WAR') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REPO_NAME}:${IMAGE_TAG}")
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
       stage('Debug Credentials') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus_login',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASS'
        )]) {
            sh 'echo "Injected user: $NEXUS_USER"'
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
                    docker tag ${REPO_NAME}:${IMAGE_TAG} ${ECR_URL}/${REPO_NAME}:${IMAGE_TAG}
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
            echo "🎉 Build + Nexus Upload + ECR Push completed successfully!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}
