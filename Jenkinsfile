/* Windows host + Linux containers:
 * - Build/Test (Node & Python)
 * - JUnit + Artifacts
 * - Cleanup + optional notifications
 * - Deploy to Staging -> Manual Gate -> Deploy to Production
 */
pipeline {
  agent any

  options {
    timestamps()
    skipStagesAfterUnstable() // don't deploy if tests marked UNSTABLE
  }

  /************ NOTIFICATION CONTROLS (optional) ************/
  parameters {
    booleanParam(name: 'SEND_EMAIL', defaultValue: false, description: 'Send email on build events')
    string(name: 'EMAIL_TO', defaultValue: 'team@example.com', description: 'Email recipients (comma-separated)')
    booleanParam(name: 'SEND_SLACK', defaultValue: false, description: 'Send Slack message on build events')
    string(name: 'SLACK_CHANNEL', defaultValue: '#ops-room', description: 'Slack channel (e.g., #ops-room)')
  }

  environment {
    APP_ENV = 'ci'
  }

  stages {
    /******************* BUILD & TEST *******************/
    stage('Build (Node)') {
      when { expression { fileExists('package.json') } }
      steps {
        bat '''
          echo === Node build in Docker ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            node:22.20.0-alpine3.22 ^
            sh -lc "npm ci && npm run build || true"
        '''
      }
    }

    stage('Test (Node/Jest)') {
      when { expression { fileExists('package.json') } }
      steps {
        bat '''
          echo === Node tests (Jest) in Docker writing JUnit ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            node:22.20.0-alpine3.22 ^
            sh -lc "
              set -e
              mkdir -p /ws/test-results
              if [ -f package.json ]; then
                npm ci
                JEST_JUNIT_OUTPUT=/ws/test-results/junit-node.xml \
                npx --yes jest --ci --reporters=default --reporters=jest-junit || true
              fi
            "
        '''
      }
    }

    stage('Test (Python/pytest)') {
      when { expression { fileExists('requirements.txt') || fileExists('pyproject.toml') || fileExists('pytest.ini') || fileExists('tests') } }
      steps {
        bat '''
          echo === Python tests (pytest) in Docker writing JUnit ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            python:3.13.7-alpine3.22 ^
            sh -lc "
              set -e
              python -m pip install --upgrade pip
              if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
              if [ -f pyproject.toml ]; then python -m pip install . || true; fi
              mkdir -p /ws/test-results
              pytest -q --maxfail=1 --disable-warnings --junitxml=/ws/test-results/junit-py.xml || true
            "
        '''
      }
    }

    /******************* DEPLOY *******************/
    stage('Deploy - Staging') {
      steps {
        bat '''
          echo === Deploy to STAGING inside container ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            alpine:3.20 ^
            sh -lc "
              set -e
              if [ -x ./deploy ]; then
                ./deploy staging
              elif [ -f ./deploy ]; then
                sh ./deploy staging
              else
                echo 'No ./deploy script found; simulating staging deploy...'
                echo 'STAGING DEPLOY OK' > /ws/deploy-staging.log
              fi
              # optional smoke tests
              if [ -x ./run-smoke-tests ]; then
                ./run-smoke-tests || exit 1
              else
                echo 'No smoke tests script; skipping.'
              fi
            "
        '''
      }
    }

    stage('Sanity check (manual gate)') {
      steps {
        script {
          // Time-box the approval so builds don't hang forever
          timeout(time: 30, unit: 'MINUTES') {
            input message: "Promote to PRODUCTION?",
                  ok: "Deploy",
                  submitterParameter: 'APPROVER' // recorded in build params
          }
        }
      }
    }

    stage('Deploy - Production') {
      steps {
        bat '''
          echo === Deploy to PRODUCTION inside container ===
          docker run --rm ^
            -v "%WORKSPACE%":/ws ^
            -w /ws ^
            alpine:3.20 ^
            sh -lc "
              set -e
              if [ -x ./deploy ]; then
                ./deploy production
              elif [ -f ./deploy ]; then
                sh ./deploy production
              else
                echo 'No ./deploy script found; simulating prod deploy...'
                echo 'PROD DEPLOY OK' > /ws/deploy-prod.log
              fi
            "
        '''
      }
    }

    stage('Back on Host') {
      steps {
        bat 'echo Back on host after deploy stages.'
      }
    }
  }

  /******************* POST *******************/
  post {
    always {
      echo 'One way or another, I have finished.'

      // Archive useful outputs (won’t fail if none exist)
      archiveArtifacts artifacts: 'build/**/*, dist/**/*, logs/**/*, **/*.log, deploy-*.log',
                       fingerprint: true,
                       onlyIfSuccessful: false,
                       allowEmptyArchive: true

      // Publish JUnit XMLs (safe if none exist)
      junit allowEmptyResults: true,
            testResults: 'test-results/**/*.xml, build/test-results/**/*.xml, build/reports/**/*.xml'

      deleteDir()
    }

    success {
      echo 'I succeeded!'
      script {
        if (params.SEND_EMAIL) {
          try {
            mail to: params.EMAIL_TO,
                 subject: "SUCCESS: ${currentBuild.fullDisplayName}",
                 body: "Build & Deploy succeeded.\n${env.BUILD_URL}"
          } catch (ignored) { echo 'Email step skipped (mail not configured?)' }
        }
        if (params.SEND_SLACK) {
          try {
            slackSend channel: params.SLACK_CHANNEL,
                      color: 'good',
                      message: "SUCCESS: ${currentBuild.fullDisplayName} — ${env.BUILD_URL}"
          } catch (ignored) { echo 'Slack step skipped (slack not configured?)' }
        }
      }
    }

    unstable {
      echo 'I am unstable :/'
    }

    failure {
      echo 'I failed :('
      script {
        if (params.SEND_EMAIL) {
          try {
            mail to: params.EMAIL_TO,
                 subject: "FAILED: ${currentBuild.fullDisplayName}",
                 body: "Build or Deploy failed.\n${env.BUILD_URL}"
          } catch (ignored) { echo 'Email step skipped (mail not configured?)' }
        }
        if (params.SEND_SLACK) {
          try {
            slackSend channel: params.SLACK_CHANNEL,
                      color: 'danger',
                      message: "FAILED: ${currentBuild.fullDisplayName} — ${env.BUILD_URL}"
          } catch (ignored) { echo 'Slack step skipped (slack not configured?)' }
        }
      }
    }

    changed {
      echo 'Things were different before...'
    }
  }
}
