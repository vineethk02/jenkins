pipeline {
  agent any
  options { timestamps() }

  parameters {
    // Leave default empty; set in job config if/when you add DockerHub creds
    string(name: 'DOCKERHUB_CREDS_ID', defaultValue: '', description: 'Credentials ID for DockerHub (username/password). Leave empty to skip.')
  }

  environment {
    DISABLE_AUTH = 'true'
    DB_ENGINE    = 'sqlite'
    APP_ENV      = 'ci'
    // Bind Secret Text credential to env var (requires a credential with this ID)
    API_TOKEN    = credentials('my-api-token')
  }

  stages {
    stage('Show env on Host (Windows)') {
      steps {
        bat '''
          echo === Host Environment (Windows) ===
          echo DB_ENGINE=%DB_ENGINE%
          echo DISABLE_AUTH=%DISABLE_AUTH%
          echo APP_ENV=%APP_ENV%
          rem NOTE: API_TOKEN is masked and should not be echoed.
        '''
      }
    }

    stage('Use env in Node container') {
      steps {
        bat '''
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            -e DB_ENGINE="%DB_ENGINE%" ^
            -e DISABLE_AUTH="%DISABLE_AUTH%" ^
            -e APP_ENV="%APP_ENV%" ^
            -e API_TOKEN="%API_TOKEN%" ^
            node:22.20.0-alpine3.22 ^
            sh -lc "echo DB_ENGINE=$DB_ENGINE && echo DISABLE_AUTH=$DISABLE_AUTH && echo APP_ENV=$APP_ENV && test -n \\"$API_TOKEN\\" && echo TOKEN_PRESENT=1 || echo TOKEN_PRESENT=0"
        '''
      }
    }

    stage('Optional: DockerHub login + curl with token') {
      when {
        expression { return params.DOCKERHUB_CREDS_ID?.trim() }
      }
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: params.DOCKERHUB_CREDS_ID, usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
            bat '''
              docker run --rm ^
                -v "%WORKSPACE%":/ws ^
                -w /ws ^
                -e API_TOKEN="%API_TOKEN%" ^
                -e DH_USER="%DH_USER%" ^
                -e DH_PASS="%DH_PASS%" ^
                curlimages/curl:8.11.1 ^
                sh -lc "echo Using masked creds; curl -sS -H \\"Authorization: Bearer $API_TOKEN\\" https://httpbin.org/bearer >/dev/null || true"
            '''
          }
        }
      }
    }

    stage('Back on Host') {
      steps {
        bat 'echo Back on host after container stages. & dir'
      }
    }
  }

  post {
    always { cleanWs() }
  }
}
