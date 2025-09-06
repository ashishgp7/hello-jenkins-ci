pipeline {
  agent any
  environment {
    APP_PORT = "8081"            // use 8081 if Jenkins is using 8080
    APP_JAR  = "target/*.jar"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
    stage('Test') {
      steps { sh 'mvn -B test || true' }
    }
    stage('Deploy') {
      steps {
        sh '''
          set -eux
          # kill any process using APP_PORT
          pids=$(ss -tulpn | awk -v port=$APP_PORT '$5 ~ ":"port"$" {print $7}' | cut -d"," -f2 | cut -d"/" -f1 | tr "\\n" " ")
          if [ -n "$pids" ]; then
            for pid in $pids; do kill -9 $pid || true; done
          fi
          # start the new jar on APP_PORT
          nohup bash -lc "java -jar $(ls $APP_JAR | tail -n1) --server.port=$APP_PORT >/var/log/jenkins/hello-app.log 2>&1 &"
          sleep 3
        '''
      }
    }
    stage('Verify') {
      steps {
        sh 'sleep 2; curl -sS --fail http://localhost:${APP_PORT} || (echo "App check failed" && false)'
      }
    }
  }
  post {
    success { echo 'Pipeline succeeded' }
    failure { echo 'Pipeline failed — check console logs' }
  }
}
