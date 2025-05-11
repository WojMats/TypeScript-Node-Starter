pipeline {
  agent any

  environment {
    // tagi do obrazów
    NODE_IMAGE   = 'node:18-alpine'
    APP_IMAGE    = "ts-node-starter:ci-${env.BUILD_NUMBER}"
    MONGO_IMAGE  = 'mongo:4.4'
    NETWORK      = 'ci-net'
  }

  stages {
    stage('Prepare network') {
      steps {
        // utworzenie sieci, jeśli już istnieje to nic się nie stanie
        sh "docker network create ${NETWORK} || true"
      }
    }

    stage('Build app image') {
      steps {
        // używamy Dockerfile.builder jeśli masz oddzielny, albo Dockerfile
        sh "docker build --pull -t ${APP_IMAGE} ."
      }
    }

    stage('Start Mongo') {
      steps {
        // uruchamiamy mongo w tle
        sh "docker run -d --name ci-mongo --network ${NETWORK} ${MONGO_IMAGE}"
      }
    }

    stage('Test') {
      steps {
        // uruchamiamy kontener z naszą aplikacją i odpalamy testy
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
      // dla pewności posprzątaj
      sh "docker rm -f ci-mongo || true"
      sh "docker network rm ${NETWORK} || true"
    }
  }
}
