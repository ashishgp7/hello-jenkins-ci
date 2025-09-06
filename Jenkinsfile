pipeline {
  agent any

  environment {
    APP_DIR  = "complete"
    APP_PORT = "8081"
    APP_JAR  = "${env.APP_DIR}/target/*.jar"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        dir("${APP_DIR}") {
          sh 'mvn -B -DskipTests package'
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Test') {
      steps {
        dir("${APP_DIR}") {
          sh 'mvn -B test || true'
        }
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -eux
          # stop any process using APP_PORT
          pids=$(ss -tulpn | awk -v port=${APP_PORT} '$5 ~ ":"port"$" {print $7}' | cut -d"," -f2 | cut -d"/" -f1 | tr "\\n" " ")
          if [ -n "$pids" ]; then
            for pid in $pids; do kill -9 $pid || true; done
          fi
          jar=$(ls ${APP_DIR}/target/*.jar | tail -n1)
          nohup bash -lc "java -jar \"$jar\" --server.port=${APP_PORT} >/var/log/jenkins/hello-app.log 2>&1 &" &
          sleep 5
        '''
      }
    }

    stage('Verify') {
      steps {
        sh "sleep 2; curl -sS --fail http://localhost:${APP_PORT} || (echo 'App check failed' && false)"
      }
    }
  }

  post {
    success { echo 'Pipeline succeeded' }
    failure { echo 'Pipeline failed — check console logs' }
  }
}
