pipeline {
  agent any

  environment {
    NODE_IMAGE   = 'node:18-alpine'
    APP_IMAGE    = "ts-node-starter:ci-${env.BUILD_NUMBER}"
    MONGO_IMAGE  = 'mongo:4.4'
    NETWORK      = 'ci-net'
  }

  stages {
    stage('Prepare network') {
      steps {
        sh "docker network create ${NETWORK} || true"
      }
    }

    stage('Build app image') {
      steps {
        // Budujemy obraz z pliku Dockerfile.jenkins
        sh "docker build --pull -t ${APP_IMAGE} -f Dockerfile.jenkins ."
      }
    }

    stage('Start Mongo') {
      steps {
        sh "docker run -d --name ci-mongo --network ${NETWORK} ${MONGO_IMAGE}"
      }
    }

    stage('Test') {
      steps {
        sh """
          docker run --rm \
            --network ${NETWORK} \
            -e MONGODB_URI_LOCAL="mongodb://ci-mongo:27017/express-typescript-starter" \
            ${APP_IMAGE} \
            npm test
        """
      }
    }

    stage('Cleanup Mongo') {
      steps {
        sh "docker rm -f ci-mongo || true"
      }
    }

    stage('Publish artifacts') {
      steps {
        archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
      }
    }
  }

  post {
    always {
      sh "docker rm -f ci-mongo || true"
      sh "docker network rm ${NETWORK} || true"
    }
  }
}

