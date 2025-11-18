pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/rahul727801/Demo2.git'
            }
        }

        stage('Build') {
            steps {
                echo "Running Build Stage..."
                sh 'echo Building App...'
            }
        }

        stage('Test') {
            steps {
                echo "Running Test Stage..."
                sh 'echo Running Tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo "Running Deploy Stage..."
                sh 'echo Deploying App...'
            }
        }
    }
}

