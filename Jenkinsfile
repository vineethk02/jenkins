/* Windows host + Linux containers:
 * - Collect JUnit test reports
 * - Archive build artifacts
 * - Works whether you have Node, Python, or both
 */
pipeline {
  agent any
  options {
    timestamps()
    // Stop later stages after UNSTABLE tests (optional; remove if you want to continue)
    skipStagesAfterUnstable()
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
      // Archive typical build outputs (won’t fail if none are present)
      archiveArtifacts artifacts: 'build/**/*, dist/**/*, logs/**/*, **/*.log',
                       fingerprint: true,
                       onlyIfSuccessful: false,
                       allowEmptyArchive: true

      // Publish all JUnit XMLs we may have produced
      junit allowEmptyResults: true, testResults: 'test-results/**/*.xml, build/test-results/**/*.xml, build/reports/**/*.xml'

      cleanWs()
    }
  }
}
