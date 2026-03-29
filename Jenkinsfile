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

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "🎉 Build completed successfully!"
        }
        failure {
            echo "❌ Build failed!"
        }
    }
}
