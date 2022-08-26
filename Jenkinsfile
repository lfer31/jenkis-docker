stage('build') {
  node {
    def app

    stage('Clone repository') {
      checkout scm
    }

    stage('Build image') {
      app = docker.build("flora-test:5001/jenkins-docker")
    }

    stage('Test image') {
      app.inside {
        sh 'echo "Tests passed"'
      }
      sh 'docker info | grep -A 20 "Insecure Registries"'
    }

    stage('Push image') {
      docker.withRegistry('http://flora-test:5000', '') {
        app.push("v${env.BUILD_NUMBER}")
      }
    }
  }
}

pipeline {

  options {
    ansiColor('xterm')
  }

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {

    stage('Deploy App to Kubernetes') {
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/v${BUILD_NUMBER}/" deploy-jenkins.yml'
            sh 'kubectl apply -f deploy-jenkins.yml'
          }
        }
      }
    }

  }
}
