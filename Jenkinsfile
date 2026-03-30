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
                    dockerImage = docker.build("vprofile-app:${BUILD_NUMBER}")
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

