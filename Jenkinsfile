/* Windows host + Linux containers:
 * - JUnit + Artifacts
 * - Cleanup + Notifications (email/slack optional)
 */
pipeline {
  agent any

  options {
    timestamps()
    // Skip later stages when tests mark build UNSTABLE
    skipStagesAfterUnstable()
  }

  /************ NOTIFICATION CONTROLS ************/
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

    stage('(Optional) Package Artifacts') {
      when { anyOf { expression { fileExists('build') }; expression { fileExists('dist') } } }
      steps {
        bat 'echo Packaging done (if any).'
      }
    }

    stage('Back on Host') {
      steps {
        bat 'echo Back on host after container stages.'
      }
    }
  }

  post {
    always {
      echo 'One way or another, I have finished.'

      // Archive typical outputs (won’t fail if none exist)
      archiveArtifacts artifacts: 'build/**/*, dist/**/*, logs/**/*, **/*.log',
                       fingerprint: true,
                       onlyIfSuccessful: false,
                       allowEmptyArchive: true

      // Publish all JUnit XMLs (safe if none exist)
      junit allowEmptyResults: true,
            testResults: 'test-results/**/*.xml, build/test-results/**/*.xml, build/reports/**/*.xml'

      // Clean workspace (cross-platform)
      deleteDir()
    }

    success {
      echo 'I succeeded!'
      script {
        if (params.SEND_EMAIL) {
          try {
            mail to: params.EMAIL_TO,
                 subject: "SUCCESS: ${currentBuild.fullDisplayName}",
                 body: "Build succeeded.\n${env.BUILD_URL}"
          } catch (ignored) { echo 'Email step skipped (mail plugin not configured?)' }
        }
        if (params.SEND_SLACK) {
          try {
            slackSend channel: params.SLACK_CHANNEL,
                      color: 'good',
                      message: "SUCCESS: ${currentBuild.fullDisplayName} — ${env.BUILD_URL}"
          } catch (ignored) { echo 'Slack step skipped (slack plugin not configured?)' }
        }
      }
    }

    unstable {
      echo 'I am unstable :/'
      script {
        if (params.SEND_EMAIL) {
          try {
            mail to: params.EMAIL_TO,
                 subject: "UNSTABLE: ${currentBuild.fullDisplayName}",
                 body: "Some tests failed.\n${env.BUILD_URL}"
          } catch (ignored) { echo 'Email step skipped (mail plugin not configured?)' }
        }
        if (params.SEND_SLACK) {
          try {
            slackSend channel: params.SLACK_CHANNEL,
                      color: 'warning',
                      message: "UNSTABLE: ${currentBuild.fullDisplayName} — ${env.BUILD_URL}"
          } catch (ignored) { echo 'Slack step skipped (slack plugin not configured?)' }
        }
      }
    }

    failure {
      echo 'I failed :('
      script {
        if (params.SEND_EMAIL) {
          try {
            mail to: params.EMAIL_TO,
                 subject: "FAILED: ${currentBuild.fullDisplayName}",
                 body: "Something went wrong.\n${env.BUILD_URL}"
          } catch (ignored) { echo 'Email step skipped (mail plugin not configured?)' }
        }
        if (params.SEND_SLACK) {
          try {
            slackSend channel: params.SLACK_CHANNEL,
                      color: 'danger',
                      message: "FAILED: ${currentBuild.fullDisplayName} — ${env.BUILD_URL}"
          } catch (ignored) { echo 'Slack step skipped (slack plugin not configured?)' }
        }
      }
    }

    changed {
      echo 'Things were different before...'
    }
  }
}
