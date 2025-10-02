/* Guided Tour: Full Docker-based Pipeline
 * Requires: Docker Pipeline plugin
 */
pipeline {
  agent { docker { image 'python:3.13.7-alpine3.22' } }
  options { timestamps() }

  environment {
    APP_ENV = 'dev'
  }

  stages {
    stage('Create your first Pipeline') {
      steps {
        sh 'echo "Hello from Jenkins inside Docker!"'
      }
    }

    stage('Run multiple steps') {
      steps {
        sh '''
          echo "[Build] Compiling (simulated)…"
          echo "[Test] Quick checks…"
          echo "[Package] Creating build output…"
          mkdir -p build
          echo "demo" > build/app.txt
          ls -la build
        '''
      }
    }

    stage('Defining execution environments') {
      steps {
        sh '''
          echo "[Env] Printing container/runtime details"
          python --version
          uname -a
          cat /etc/os-release | head -n 1
        '''
      }
    }

    stage('Using environment variables') {
      steps {
        sh 'echo "Build number: $BUILD_NUMBER in $APP_ENV (workspace: $WORKSPACE)"'
      }
    }

    stage('Recording tests and artifacts') {
      steps {
        // Minimal JUnit report with two passing tests
        sh 'printf "%s" "<?xml version=\\"1.0\\" encoding=\\"UTF-8\\"?><testsuite name=\\"demo\\" tests=\\"2\\" failures=\\"0\\"><testcase classname=\\"demo\\" name=\\"t1\\"/><testcase classname=\\"demo\\" name=\\"t2\\"/></testsuite>" > test-results.xml'
        junit 'test-results.xml'
        archiveArtifacts artifacts: 'build/**/*, test-results.xml', fingerprint: true, onlyIfSuccessful: false
      }
    }

    stage('Deployment') {
      steps {
        sh '''
          echo "[Deploy] Simulating deploy…"
          mkdir -p deploy
          cp -f build/app.txt deploy/app.txt
          ls -la deploy
        '''
        echo 'Deployment simulated.'
      }
    }
  }

  post {
    success {
      echo '✅ Pipeline completed successfully.'
      echo 'Notifying team (simulated).'
    }
    failure {
      echo '❌ Build failed. (Simulated notification would fire here.)'
    }
    always {
      cleanWs()  // Cleanup workspace after archiving artifacts
    }
  }
}
