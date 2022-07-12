pipeline {
    agent {label 'zuerh407'}

    options {

        disableConcurrentBuilds()
        buildDiscarder(logRotator(artifactDaysToKeepStr: '7', artifactNumToKeepStr: '1',
                                  daysToKeepStr: '7', numToKeepStr: '1'))
        timeout(time: 48, unit: 'HOURS')
    }

   
    stages {
        stage('Checkout') {
            environment {
                IMAGE = "docker-intern-nexus.meteoswiss.ch/test"
                DOCKER_CONFIG = "$workspace/.docker"
                HTTP_PROXY = 'http://proxy.meteoswiss.ch:8080/'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'openshift-nexus',
                                          passwordVariable: 'NXPASS',
                                          usernameVariable: 'NXUSER')]) {
                    sh """
                        echo \$NXPASS | docker login docker-all-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        echo \$NXPASS | docker login docker-intern-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        echo "proxy $http_proxy"
                        docker build --pull .
                    """
                              }
            }
        }
    }
    post {
        always {
            echo "Build stage complete"
        }
        failure {
            echo "Build failed"
            emailext(subject: "${currentBuild.fullDisplayName}: ${currentBuild.currentResult}",
                     body: """Job '${env.JOB_NAME} #${env.BUILD_NUMBER}': ${env.BUILD_URL}""",
                     recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']])
        }
        success {
            echo "Build succeeded"
        }
    }
}
