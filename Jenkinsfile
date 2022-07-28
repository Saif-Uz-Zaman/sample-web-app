pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
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

    stage('string (secret text)') {
      steps {
        script {
          withCredentials([
            string(
              credentialsId: 'kube_config',
              variable: 'joke')
          ]) {
            sh 'echo $joke'
            sh 'echo $joke > sample.txt'
            sh 'cat sample.txt'
            print 'joke=' + joke
            print 'joke.collect { it }=' + joke.collect { it }
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