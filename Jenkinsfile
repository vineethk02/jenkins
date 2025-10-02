pipeline {
  agent any
  options {
    timestamps()
    disableConcurrentBuilds()
    timeout(time: 10, unit: 'MINUTES') // global safety net
  }
  environment { APP_ENV = 'dev' }

  stages {
    stage('Create your first Pipeline') {
      steps {
        echo 'Hello from Jenkins!'
      }
    }

    stage('Run multiple steps (Windows)') {
      steps {
        // single command
        bat 'echo "Hello World"'

        // multi-line batch (like the docs' multi-line sh)
        bat '''
          echo "Multiline batch steps work too"
          dir
        '''
      }
    }

    stage('Using environment variables') {
      steps {
        echo "Build number: ${env.BUILD_NUMBER} in ${env.APP_ENV}"
      }
    }

    stage('Build & Package (example)') {
      steps {
        bat '''
          echo [Build] Compiling (simulated)...
          if not exist build mkdir build
          echo demo> build\\app.txt
        '''
      }
    }

    stage('Recording tests and artifacts') {
      steps {
        // tiny JUnit report with 2 passing tests
        writeFile file: 'test-results.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="demo" tests="2" failures="0">
  <testcase classname="demo" name="t1"/>
  <testcase classname="demo" name="t2"/>
</testsuite>'''
        junit testResults: 'test-results.xml', keepLongStdio: true
        archiveArtifacts artifacts: 'build/**/*, test-results.xml', fingerprint: true
      }
    }

    stage('Deploy (with retry + timeout)') {
      steps {
        // create a flaky deploy script to demonstrate retry
        writeFile file: 'flakey-deploy.cmd', text: '''
@echo off
REM Simulate a flaky deployment: randomly fail ~50%% of the time
set /a R=%RANDOM% %% 2
if %R%==0 (
  echo Deploy step failed randomly.
  exit /b 1
) else (
  echo Deploy step succeeded.
  exit /b 0
)
'''
        // Retry up to 5 times but never spend more than 3 minutes total
        timeout(time: 3, unit: 'MINUTES') {
          retry(5) {
            bat 'call flakey-deploy.cmd'
          }
        }

        // Quick "health check" with its own timeout (like the docs)
        timeout(time: 1, unit: 'MINUTES') {
          bat 'echo Health check OK'
        }

        // Simulated deployment artifact
        bat '''
          if not exist deploy mkdir deploy
          copy /Y build\\app.txt deploy\\app.txt >NUL
        '''
        archiveArtifacts artifacts: 'deploy/app.txt', fingerprint: true, allowEmptyArchive: true
      }
    }
  }

  post {
    always {
      echo 'This will always run'
      cleanWs()
    }
    success {
      echo 'This will run only if successful'
    }
    failure {
      echo 'This will run only if failed'
    }
    unstable {
      echo 'This will run only if unstable'
    }
    changed {
      echo 'Runs only if the build status changed since last run'
    }
  }
}
