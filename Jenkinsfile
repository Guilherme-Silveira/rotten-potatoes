pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
            containers:
            - name: docker
              image: docker:dind
              securityContext:
                privileged: true
              env:
              - name: DOCKER_TLS_CERTDIR
                value: ""
            - name: kubectl
              image: guisilveira/kubectl
              command:
                - cat
              tty: true 
      '''
    }
  }

  stages {
    stage('git checkout') {
      steps {
        container('docker') {
          git url: 'https://github.com/Guilherme-Silveira/rotten-potatoes.git', branch: 'main'
        }
      }
    }
    stage('docker build') {
      steps {
        container('docker') {
          script {
            dockerapp = docker.build("guisilveira/rotten-potatoes:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
          }
        }
      }
    }
    stage('docker push') {
      steps {
        container('docker') {
          script {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
              dockerapp.push('latest')
              dockerapp.push("${env.BUILD_ID}")
            }
          }
        }
      }
    }
    stage('deploy on kubernetes') {
      environment {
        tag_version = "${env.BUILD_ID}"
      }
      steps {
        container('kubectl') {
          git url: 'https://github.com/Guilherme-Silveira/rotten-potatoes.git', branch: 'main'
          withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://k3d-k3s-default-server-0:6443']) {
            sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
            sh 'kubectl apply -f ./k8s'
          }
        }
      }
    }
  }
}
