/* Defining execution environments on Windows with per-stage Docker containers
 * Requires: Docker Desktop + Docker Pipeline plugin
 */
pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Host (Windows)') {
      steps {
        bat '''
          ver
          echo Host workspace: %CD%
        '''
      }
    }

    stage('Node in Docker') {
      steps {
        script {
          // Mount the Windows workspace into /ws and use a POSIX workdir
          def args = "-v \"${env.WORKSPACE}\":/ws -w /ws"
          docker.image('node:22.20.0-alpine3.22').inside(args) {
            sh 'node --eval "console.log(process.arch, process.platform)"'
            sh 'echo "Listing container filesystem:" && ls -lah'
          }
        }
      }
    }

    stage('Python in Docker') {
      steps {
        script {
          def args = "-v \"${env.WORKSPACE}\":/ws -w /ws"
          docker.image('python:3.13.7-alpine3.22').inside(args) {
            sh 'python --version'
          }
        }
      }
    }

    stage('Back on Host') {
      steps {
        bat '''
          echo Back on host after container stages.
          dir
        '''
      }
    }
  }

  post {
    always { cleanWs() }
  }
}
