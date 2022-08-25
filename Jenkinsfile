pipeline {

    options {
      ansiColor('xterm')
    }

    environment {
      app = ""
    }

    agent {
      kubernetes {
        yamlFile 'builder.yaml'
      }
    }

    stages {
      stage('Clone repository') {
        steps {
          checkout scm
        }
      }

      stage('Build image') {
        steps {
          script {
            app = docker.build("flora-test:5000/jenkins-docker:v${env.BUILD_NUMBER}")
          }
        }
      }

      stage('Test image') {
        steps {
          script {
            app.inside {
              sh 'echo "Tests passed"'
            }
            sh 'docker info | grep -A 20 "Insecure Registries"'
          }
        }
      }

      stage('Push image') {
        steps {
          script {
            docker.withRegistry('http://flora-test:5000', '') {
              app.push("v${env.BUILD_NUMBER}")
            }
          }
        }
      }

      stage('Deploy App to Kubernetes') {
        steps {
          container('kubectl') {
            withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
              sh 'sed -i "s/<TAG>/v${BUILD_NUMBER}/" deploy-jenkins.yaml'
              sh 'kubectl apply -f deploy-jenkins.yaml'
            }
          }
        }
      }
    }
}
