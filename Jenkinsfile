// Declarative, multi-language Jenkinsfile that auto-detects Maven / Node / Docker projects.
// Customize tool labels and credential IDs to match your Jenkins configuration.

pipeline {
  agent any

  

  environment {
    // Example env vars - change or remove as needed
    IMAGE_NAME = "myorg/demo2"           // change if building Docker images    DOCKER_REGISTRY = "registry.hub.docker.com"
    // CREDENTIAL IDS: create credentials in Jenkins and replace these IDs
    DOCKER_CREDENTIALS = 'docker-creds-id'
    MAVEN_SETTINGS_CREDENTIALS = 'maven-settings-id' // optional for private repos
  }

  options {
    // Keep only last 10 builds; adjust as needed
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          echo "Detected files at workspace root:"
          sh 'ls -la || true'
        }
      }
    }

    stage('Detect & Setup') {
      steps {
        script {
          // simple detection
          hasPom = fileExists('pom.xml')
          hasPkg = fileExists('package.json')
          hasDockerfile = fileExists('Dockerfile') || fileExists('docker/Dockerfile')

          echo "hasPom=${hasPom}, hasPkg=${hasPkg}, hasDockerfile=${hasDockerfile}"
        }
      }
    }

    stage('Build - Maven') {
      when {
        expression { return fileExists('pom.xml') }
      }
      steps {
        script {
          // If you need a specific settings.xml stored in Jenkins credentials, use withCredentials (example commented)
          // withCredentials([file(credentialsId: env.MAVEN_SETTINGS_CREDENTIALS, variable: 'MAVEN_SETTINGS_XML')]) {
          //   sh 'mvn -s $MAVEN_SETTINGS_XML -B -DskipTests clean package'
          // }
          sh 'mvn -B -DskipTests clean package'
        }
      }
      post {
        always {
          junit '**/target/surefire-reports/*.xml, **/target/failsafe-reports/*.xml'
          archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
        }
      }
    }

    stage('Test - Maven (unit)') {
      when {
        expression { return fileExists('pom.xml') }
      }
      steps {
        sh 'mvn -B test'
      }
      post {
        always {
          junit '**/target/surefire-reports/*.xml'
        }
      }
    }

    stage('Build - Node') {
      when {
        expression { return fileExists('package.json') }
      }
      steps {
        script {
          // If you configured NodeJS tool in Jenkins, you can wrap with 'tool' steps, or ensure node/npm are on PATH.
          sh 'npm ci'
          // common build script
          sh 'if npm run | grep -q \" build\"; then npm run build; else echo \"No build script\"; fi'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
        }
      }
    }

    stage('Test - Node (unit)') {
      when {
        expression { return fileExists('package.json') }
      }
      steps {
        sh 'if npm run | grep -q \" test\"; then npm test || true; else echo \"No test script\"; fi'
      }
    }

    stage('Build Docker image') {
      when {
        expression { return fileExists('Dockerfile') || fileExists('docker/Dockerfile') }
      }
      steps {
        script {
          // Build docker image and tag with build number
          def img = "${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
          echo "Building Docker image ${img}"
          // Use docker.build if Docker pipeline plugin is available
          // def built = docker.build(img)
          // built.inside { echo "image available inside container" }
          sh "docker build -t ${img} ."
          // Save image name to env for later push stage
          env.BUILT_IMAGE = img
        }
      }
    }

    stage('Publish Docker image') {
      when {
        allOf {
          expression { return (fileExists('Dockerfile') || fileExists('docker/Dockerfile')) }
          expression { return env.DOCKER_CREDENTIALS != null && env.DOCKER_CREDENTIALS != '' }
        }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh '''
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${DOCKER_REGISTRY}
              docker push ${BUILT_IMAGE}
              docker logout ${DOCKER_REGISTRY}
            '''
          }
        }
      }
    }

    stage('Archive & Publish') {
      steps {
        script {
          // Archive common artifacts
          if (fileExists('pom.xml')) {
            archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
          }
          if (fileExists('package.json')) {
            archiveArtifacts artifacts: 'dist/**', allowEmptyArchive: true
          }
        }
      }
    }
  } // stages

  post {
    success {
      echo "Build succeeded: ${env.BUILD_URL}"
    }
    failure {
      echo "Build failed: ${env.BUILD_URL}"
    }
    always {
      // Keep logs and any other cleanup
      echo "Cleaning workspace"
      cleanWs()
    }
  }
}
