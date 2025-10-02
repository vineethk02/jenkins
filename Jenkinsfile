pipeline {
  agent any
  options { timestamps() }
  environment { APP_ENV = 'dev' }

  stages {
    stage('Create your first Pipeline') { steps { echo 'Hello from Jenkins!' } }

    stage('Run multiple steps') {
      steps {
        bat '''
          echo [Build] Compiling (simulated)...
          echo [Test] Quick checks...
          if not exist build mkdir build
          echo demo> build\\app.txt
        '''
      }
    }

    stage('Using environment variables') {
      steps { echo "Build number: ${env.BUILD_NUMBER} in ${env.APP_ENV}" }
    }

    stage('Recording tests and artifacts') {
      steps {
        writeFile file: 'test-results.xml', text: '''<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="demo" tests="2" failures="0">
  <testcase classname="demo" name="t1"/>
  <testcase classname="demo" name="t2"/>
</testsuite>'''
        junit 'test-results.xml'
        archiveArtifacts artifacts: 'build/**/*, test-results.xml', fingerprint: true
      }
    }

    stage('Deployment') {
      steps {
        bat '''
          echo [Deploy] Copying files (simulated)...
          if not exist deploy mkdir deploy
          copy /Y build\\app.txt deploy\\app.txt >NUL
        '''
        echo 'Deployment simulated.'
      }
    }
  }

  post {
    success { echo '✅ Pipeline completed successfully.' }
    failure { echo '❌ Build failed.' }
    always  { cleanWs() }
  }
}
