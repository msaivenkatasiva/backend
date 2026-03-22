// pipeline {
//     agent {
//         label 'AGENT-1'
//     }
//     options {
//         timeout(time: 60, unit: 'MINUTES')
//         disableConcurrentBuilds()
//         ansiColor('xterm')
//     }
//     environment {
//         def appVersion = '' //variable declaration
//         def nexusUrl = 'nexus.devops76.sbs:8081'
//     }
//     parameters{
//         booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
//     }
    
//     stages {
//         stage('read the version') {
//             steps {
//                 script {
//                 def packageJson = readJSON file: 'package.json'
//                 appVersion = packageJson.version
//                 echo "application version: $appVersion"
//                 }
//             }
//         }

//         stage('install dependencies') {
//             steps {
//                 sh """
//                  npm install
//                  ls -ltr
//                  echo "application version: $appVersion" 
//                 """
//             }
//         }
//         stage('Build'){
//             steps {
//                 sh """
//                 zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
//                 ls -ltr
//                 """
//             }
//         }

//         stage('sonar scan'){
//             environment {
//                 scannerHome = tool 'sonar-7.0' //refering scanner CLI
//             }
//             steps {
//                 script {
//                     withSonarQubeEnv('sonar-7.0') { //referring sonar server
//                         sh "${scannerHome}/bin/sonar-scanner"
//                     }
//                 }
//             }
//         }

//         // stage("Quality Gate") {
//         //     steps {
//         //       timeout(time: 30, unit: 'MINUTES') {
//         //         waitForQualityGate abortPipeline: true
//         //       }
//         //     }
//         // }

//         stage('Nexus Artifact Upload'){
//             steps {
//                 script{
//                     nexusArtifactUploader(
//                         nexusVersion: 'nexus3',
//                         protocol: 'http',
//                         nexusUrl: "${nexusUrl}",
//                         groupId: 'com.expense',
//                         version: "${appVersion}",
//                         repository: "backend",
//                         credentialsId: 'nexus-auth',
//                         artifacts: [
//                             [artifactId: "backend",
//                             classifier: '',
//                             file: 'backend-' + "${appVersion}" + '.zip',
//                             type: 'zip']
//                         ]
//                     )
//                 }
//             }
//         }
//         stage('Deploy') {
//             when{
//                 expression{
//                     params.deploy
//                 }
//             }
//             steps{
//                 script{
//                     def params = [
//                     string(name: 'appVersion', value: "${appVersion}")
//                     ]
//                     build job: 'backend-deploy', parameters: params, propagate: false
//                 } 
//             }
//         }
//     }
//     post {
//         always {
//             echo 'I Will run always'
//             deleteDir()
//         }
//         success {
//             echo 'I will run when pipeline is success'
//         }
//         failure {
//             echo 'i will run when pipeline is failure'
//         }
//     }
// }

// the below pipeline is to run ecr images to deploy and the above is to deploy with terraform through ansible config file


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
        def nexusUrl = ''
        region = "us-east-1"
        account_id = "210749645231"
    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    
    stages {
        stage('read the version') {
            steps {
                script {
                def packageJson = readJSON file: 'package.json'
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
        stage('Docker Build'){
            steps {
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion} .

                    docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion}
                """
            }
        }
        stage('Deploy'){
            steps {
                sh """
                    pwd
                    ls -R
                    
                    sed -i 's/IMAGE_VERSION/${appVersion}/g' helm/values.yaml
                    helm install backend .
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