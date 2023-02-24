pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes

  stages {
    stage('Example') {
      steps {
        sh 'mvn clean install'
        echo 'Hello World'
        }
    }
  }
}
