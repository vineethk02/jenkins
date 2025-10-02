# create a Docker-based Jenkinsfile (matches the website tour)
# Requires Docker Desktop + the "Docker Pipeline" plugin in Jenkins
@'
/* Requires the Docker Pipeline plugin */
pipeline {
  agent { docker { image 'python:3.13.7-alpine3.22' } }
  stages {
    stage('build') {
      steps {
        sh 'python --version'
      }
    }
  }
}
'@ | Out-File -Encoding utf8 Jenkinsfile
