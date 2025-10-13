pipeline {
  agent { label 'AGENT-1' }
  options {
    timeout(time: 60, unit: 'MINUTES')
    disableConcurrentBuilds()
    ansiColor('xterm')
  }
  environment {
    APP_VERSION = ""                            // will be set after reading package.json
    NEXUS_URL   = 'nexus.devops76.sbs:8081'
  }
  stages {
    stage('Read version from package.json') {
      steps {
        script {
          def pkg = readJSON file: 'package.json'
          env.APP_VERSION = (pkg.version as String)
          echo "Application version: ${env.APP_VERSION}"
        }
      }
    }
    stage('Install dependencies') {
      steps {
        sh '''
          npm install
          echo "APP_VERSION=$APP_VERSION"
        '''
      }
    }
    stage('Build artifact (zip)') {
      steps {
        sh '''
          set -e
          ART="backend-${APP_VERSION}.zip"
          zip -q -r "$ART" . -x Jenkinsfile -x "$ART"
          ls -l "$ART"
        '''
      }
    }
    stage('Upload to Nexus') {
      steps {
        script {
          nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: env.NEXUS_URL,
            repository: 'backend',
            groupId: 'com.expense',
            version: env.APP_VERSION,
            credentialsId: 'nexus-auth',
            artifacts: [[
              artifactId: 'backend',
              classifier: '',
              file: "backend-${env.APP_VERSION}.zip",
              type: 'zip'
            ]]
          )
        }
      }
    }
    stage('Trigger downstream deploy') {
      steps {
        script {
          build job: 'backend-deploy',            // <-- adjust if your job path differs
            propagate: true,
            parameters: [
              string(name: 'appVersion', value: env.APP_VERSION)
            ]
        }
      }
    }
  }
  post {
    always  { echo 'Cleanup'; deleteDir() }
    success { echo 'Upstream succeeded' }
    failure { echo 'Upstream failed' }
  }
}