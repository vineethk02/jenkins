/* Windows host + Linux containers: call Docker directly from bat to avoid
 * docker-workflow adding a Windows -w path inside the container.
 * Requires: Docker Desktop (Linux containers).
 */
pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Host (Windows)') {
      steps {
        bat '''
          ver
          echo Host workspace: %WORKSPACE%
        '''
      }
    }

    stage('Node in Docker') {
      steps {
        // Run the Node container with a POSIX workdir and the workspace mounted at /ws
        bat '''
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            node:22.20.0-alpine3.22 ^
            sh -lc "node -v && node --eval \\"console.log(process.arch, process.platform)\\" && echo Listing /ws && ls -lah"
        '''
      }
    }

    stage('Python in Docker') {
      steps {
        bat '''
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            python:3.13.7-alpine3.22 ^
            sh -lc "python --version && python -c \\"import sys,platform; print(sys.executable); print(platform.platform())\\""
        '''
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
