pipeline {
  agent any
  // any, none, label, node, docker, dockerfile, kubernetes
  tools {
    maven 'my_maven'
  }
  
  environment {
    gitName = 'junkmm'
    gitEmail = 'kimhj4270@gmail.com'
    gitWebaddress = 'https://github.com/junkmm/sb_code.git'
    gitSshaddress = 'git@github.com:junkmm/sb_code.git'
    gitCredential = 'git_cre' // github credential 생성시의 ID
    dockerHubRegistry = 'kimhj4270/sbimage'
    dockerHubRegistryCredential = 'docker_cre' // // docker credential 생성시의 ID
  }

  stages {
    stage('Checkout Github') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: gitCredential, url: gitWebaddress]]])
      }
      post {
        failure {
          echo 'Repository clone failure'
        }
        success {
          echo 'Repository clone success'
        }
      }
    }
    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
        // maven 플러그인이 미리 설치 되어있어야 함.
      }
      post {
        failure {
          echo 'maven build failure'
        }
        success {
          echo 'maven build success'
        }
      }
    }
    stage('Docker image Build') {
      steps {
        sh "docker build -t ${dockerHubRegistry}:${currentBuild.number} ."
        sh "docker build -t ${dockerHubRegistry}:latest ."
        // oolralra/sbimage:4 이런식으로 빌드가 될것이다.
        // currentBuild.number 젠킨스에서 제공하는 빌드넘버변수.
      }
      post {
        failure {
          echo 'docker image build failure'
        }
        success {
          echo 'docker image build success'
        }
      }
    }
    stage('docker image push') {
      steps {
        withDockerRegistry(credentialsId: dockerHubRegistryCredential, url: '') {
          // withDockerRegistry : docker pipeline 플러그인 설치시 사용가능.
          // dockerHubRegistryCredential : environment에서 선언한 docker_cre  
            sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker push ${dockerHubRegistry}:latest"
        }

      }
      post {
        failure {
          echo 'docker image push failure'
          sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker image rm -f ${dockerHubRegistry}:latest"
        }
        success {
          sh "docker image rm -f ${dockerHubRegistry}:${currentBuild.number}"
          sh "docker image rm -f ${dockerHubRegistry}:latest"  
          echo 'docker image push success'
        }
      }
    }
    stage('docker container deployment') {
      steps {
        sh "docker rm -f sb"
        sh "docker run -dp 5656:8085 --name sb ${dockerHubRegistry}:${currentBuild.number}"
        
      }
      post {
        failure {
          echo 'docker container deployment failure'
          slackSend (color: '#FF0000', message: "FAILURE: docker container deployment '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
          echo 'docker container deployment success'
          slackSend (color: '#0000FF', message: "SUCCESS: docker container deployment '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
      }
    }
  }
}
