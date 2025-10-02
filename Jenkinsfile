/* Windows host + Linux containers:
 * - Global and per-stage environment variables
 * - Pass env into docker run
 * - Inject credentials as environment variables
 */
pipeline {
  agent any
  options { timestamps() }

  /************ GLOBAL ENV (applies to all stages) ************/
  environment {
    DISABLE_AUTH = 'true'
    DB_ENGINE    = 'sqlite'
    APP_ENV      = 'ci'
  }

  stages {

    stage('Show env on Host (Windows)') {
      steps {
        bat '''
          echo === Host Environment (Windows) ===
          echo DB_ENGINE=%DB_ENGINE%
          echo DISABLE_AUTH=%DISABLE_AUTH%
          echo APP_ENV=%APP_ENV%
          echo.
          rem Print all env (Windows)
          set
        '''
      }
    }

    stage('Use env in Node container') {
      steps {
        // Pass global env into the container as -e flags
        bat '''
          echo === Node in Docker with env ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            -e DB_ENGINE="%DB_ENGINE%" ^
            -e DISABLE_AUTH="%DISABLE_AUTH%" ^
            -e APP_ENV="%APP_ENV%" ^
            node:22.20.0-alpine3.22 ^
            sh -lc "echo DB_ENGINE=$DB_ENGINE && echo DISABLE_AUTH=$DISABLE_AUTH && echo APP_ENV=$APP_ENV && printenv | sort | head -n 20"
        '''
      }
    }

    /************ PER-STAGE ENV OVERRIDE ************/
    stage('Override env for this stage only') {
      environment {
        DB_ENGINE    = 'postgres' // overrides global just for this stage
        DISABLE_AUTH = 'false'
      }
      steps {
        bat '''
          echo === Per-Stage env (override) on Host ===
          echo DB_ENGINE=%DB_ENGINE%
          echo DISABLE_AUTH=%DISABLE_AUTH%
        '''
        bat '''
          echo === Per-Stage env inside Python container ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            -e DB_ENGINE="%DB_ENGINE%" ^
            -e DISABLE_AUTH="%DISABLE_AUTH%" ^
            -e APP_ENV="%APP_ENV%" ^
            python:3.13.7-alpine3.22 ^
            sh -lc "python --version && echo DB_ENGINE=$DB_ENGINE && echo DISABLE_AUTH=$DISABLE_AUTH && echo APP_ENV=$APP_ENV"
        '''
      }
    }

    /************ CREDENTIALS AS ENV VARS ************/
    stage('Credentials in the Environment') {
      steps {
        withCredentials([
          string(credentialsId: 'my-api-token', variable: 'API_TOKEN'),
          usernamePassword(credentialsId: 'my-dockerhub', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')
        ]) {
          // DO NOT echo secrets. Use them directly.
          bat '''
            echo === Using credentials inside a container (masked) ===
            docker run --rm ^
              -v "%WORKSPACE%":/ws ^
              -w /ws ^
              -e API_TOKEN="%API_TOKEN%" ^
              -e DH_USER="%DH_USER%" ^
              -e DH_PASS="%DH_PASS%" ^
              curlimages/curl:8.11.1 ^
              sh -lc "echo \\"Hitting an API with bearer token (not printed)\\" && curl -sS -H \\"Authorization: Bearer $API_TOKEN\\" https://httpbin.org/bearer || true"
          '''
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
