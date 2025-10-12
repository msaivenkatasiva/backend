pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        def appVersion = '' //variable declaration
    }
    // parameters{
    //     booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    // }
    
    stages {
        stage('read the version') {
            steps {
                script {
                def packageJson = readJson file: 'package.json'
                appVersion = packageJson.version
                echo "application version: $appVersion"
                }
            }
        }

        stage('install dependencies') {
            steps {
                sh """
                 npm install
                 ls -ltr
                 echo "application version: $appVersion" 
                """
            }
        }
        stage('Build'){
            steps {
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr
                """
            }
        }
    }
    post {
        always {
            echo 'I Will run always'
            deleteDir()
        }
        success {
            echo 'I will run when pipeline is success'
        }
        failure {
            echo 'i will run when pipeline is failure'
        }
    }
}