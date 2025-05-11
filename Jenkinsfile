pipeline {
  agent any
  environment {
    SESSION_SECRET = credentials('session-secret-id')
    MONGODB_URI   = "mongodb://mongo-test:27017/express-typescript-starter"
    IMAGE_BUILDER = "project:builder"
    IMAGE_RUNTIME = "project:runtime"
    REGISTRY      = "registry.example.com/org/project"
  }
  stages {
    stage('Collect') {
      steps { checkout scm }
    }
    stage('Build') {
      agent { docker { image 'node:18-alpine'; args '-v /var/run/docker.sock:/var/run/docker.sock' } }
      steps { sh 'docker build -f Dockerfile.builder -t $IMAGE_BUILDER .' }
    }
    stage('Test') {
      steps {
        sh 'docker network create ci-net || true'
        sh 'docker run -d --name mongo-test --network ci-net mongo:4.4'
        sh """
          docker run --rm \
            --network ci-net \
            -e MONGODB_URI=$MONGODB_URI \
            -e SESSION_SECRET=$SESSION_SECRET \
            $IMAGE_BUILDER \
            npm test
        """
      }
    }
    stage('Report') {
      steps {
        junit 'coverage/junit/*.xml'
        publishHTML target: [
          reportName: 'Coverage',
          reportDir:  'coverage/lcov-report',
          reportFiles: 'index.html'
        ]
      }
      post { always {
        sh 'docker rm -f mongo-test || true'
        sh 'docker network rm ci-net   || true'
      }}
    }
    stage('Publish') {
      when { branch 'main' }
      agent { docker { image 'docker:20.10.12-dind'; args '--privileged -v /var/run/docker.sock:/var/run/docker.sock' } }
      steps {
        sh "docker build -f Dockerfile.runtime -t $REGISTRY:latest ."
        withDockerRegistry([ credentialsId: 'docker-registry-cred', url: 'https://registry.example.com' ]) {
          sh "docker push $REGISTRY:latest"
        }
      }
    }
  }
  post {
    success { echo '✅ Pipeline zakończony sukcesem' }
    failure { echo '❌ Pipeline nie powiódł się' }
  }
}
