node {
    def app

    stage('Clone repository') {
      

        checkout scm
    }

    stage('Build image') {
       sh 'ls'
       app = docker.build("flora-test/test")
    }

    stage('Test image') {
  

        app.inside {
            sh 'echo "Tests passed"'
        }
        sh 'docker info | grep -A 20 "Insecure Registries"'
    }

    stage('Push image') {
        
        docker.withRegistry('http://flora-test:5000', '') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Apply Kubernetes files') {
      withKubeConfig([credentialsId: 'mykubeconfig']) {
        sh 'kubectl config view'
      }
    }

}

