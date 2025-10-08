pipeline {
  agent any
  options {
    timestamps()
    disableConcurrentBuilds()   // avoid port/name clashes
  }
  environment {
    APP_NAME = 'sample-app'
    PORT     = '5050'
    HOST_IP  = '192.168.56.20'          // VM private IP
    IMAGE    = "${APP_NAME}:latest"
    APP_URL      = "http://${HOST_IP}:${PORT}/"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Pre-clean') {
      steps {
        sh '''
          set -eux
          docker rm -f "$APP_NAME" 2>/dev/null || true
        '''
      }
    }
    stage('Build image') {
      steps {
        sh '''
          set -eux
          docker build -t "$IMAGE" .
        '''
      }
    }
    stage('Run container') {
      steps {
        sh '''
          set -eux
          docker run -d --name "$APP_NAME" -p "$PORT:$PORT" "$IMAGE"
        '''
      }
    }
    stage('Health check + Test') {
      steps {
        sh '''
          set -eux
          # wait until the app responds
          for i in $(seq 1 60); do
            if curl -fsS "$APP_URL" -o /tmp/resp.txt; then
              cat /tmp/resp.txt
              # Accept any valid caller IP to avoid hardcoding
              grep -Eq 'You are calling me from ([0-9]{1,3}\\.){3}[0-9]{1,3}' /tmp/resp.txt
              exit 0
            fi
            sleep 1
          done
          echo "App did not become ready in time" >&2
          docker logs "$APP_NAME" || true
          exit 1
        '''
      }
    }
  }
  post {
    always {
      sh '''
        docker ps -a || true
        docker logs "$APP_NAME" || true
      '''
    }
    cleanup {
      sh 'docker rm -f "$APP_NAME" 2>/dev/null || true'
    }
  }
}

