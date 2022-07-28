pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: kubectl
            image: dtzar/helm-kubectl:latest
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock
        '''
    }
  }
  stages {
    stage('Clone') {
      steps {
        container('docker') {
          git branch: 'master', changelog: false, poll: false, url: 'https://github.com/Saif-Uz-Zaman/sample-web-app.git'
        }
      }
    }

    stage('Build-Docker-Image') {
      steps {
        container('docker') {
          sh 'docker build -t saifmaruf/sample-web-app:$BUILD_NUMBER .'
        }
      }
    }

    stage('Login-Into-Docker') {
      steps {
        container('docker') {
          script {
            withCredentials([
              usernamePassword(credentialsId: 'docker_hub',
                usernameVariable: 'username',
                passwordVariable: 'password')
            ]) {
              sh 'docker login -u $username -p $password'
            }
          }
        }
      }
    }

    stage('Push-Images-Docker-to-DockerHub') {
      steps {
        container('docker') {
          sh 'docker push saifmaruf/sample-web-app:$BUILD_NUMBER'
        }
      }
    }

    stage('Deploy-To-Kubernetes') {
      steps {
        container('kubectl') {
          script {
            withCredentials([
              file(credentialsId: 'kube_config_file',
                variable: 'config')
            ]) {
              sh 'cp \$config /root/.kube/config'
              sh 'ls'
              sh 'kubectl get po -n jenkins'
            }
          }
        }
      }
    }   
  }
    post {
      always {
        container('docker') {
          sh 'docker logout'
      }
      }
    }
}