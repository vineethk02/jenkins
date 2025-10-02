/* Defining execution environments (host + per-stage Docker containers)
 * Requires: Docker Desktop running + Docker Pipeline plugin
 */
pipeline {
  // Default agent: run on your Windows controller/agent
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
      // This stage runs INSIDE a container
      agent {
        docker {
          image 'node:22.20.0-alpine3.22'
          // optional: args '-u root:root'  // e.g., if you need extra perms
        }
      }
      steps {
        // Linux shell inside the container
        sh 'node --eval "console.log(process.arch, process.platform)"'
        sh '''
          echo "Listing container filesystem:"
          ls -lah
        '''
      }
    }

    stage('Python in Docker') {
      agent { docker { image 'python:3.13.7-alpine3.22' } }
      steps {
        sh 'python --version'
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
