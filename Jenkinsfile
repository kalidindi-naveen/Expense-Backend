pipeline {
    agent {
        label 'naveen-2'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
      def appVersion=""
      nexusUrl = "nexus-private.step-into-iot.cloud:8081"
      repoName = "backend"
    }
    stages {
        stage('read the version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
              sh """
                npm install
                ls -ltr
                echo "application version: $appVersion"
              """
            }
        }
        stage('Build'){
            steps{
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "${repoName}",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "${repoName}" ,
                            classifier: '',
                            file: "${repoName}-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
    }
    post { 
        always { 
            echo 'Doing Cleanup........!'
            deleteDir()
        }
        success { 
            echo 'Pipeline is success........!'
        }
        failure { 
            echo 'Pipeline is failed..........!'
        }
    }
}