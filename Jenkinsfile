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
                DOCKER_CONFIG = "$workspace/.docker"
                IMAGE = "docker-intern-nexus.meteoswiss.ch/test"
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'openshift-nexus',
                                          passwordVariable: 'NXPASS',
                                          usernameVariable: 'NXUSER')]) {
                    sh """
                        echo \$NXPASS | docker login docker-all-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        echo \$NXPASS | docker login docker-intern-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        export COMPOSE_DOCKER_CLI_BUILD=1
                        export DOCKER_BUILDKIT=1
                        echo "proxy $http_proxy"
                        cp -r /etc/ssl/certs .
                        ls -ltr ~/.docker/
                        cat ~/.docker/config.json
                        docker build -v /var/run/docker.sock:/var/run/docker.sock:rw -v $HOME/.ssh:/root/.ssh:ro .
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
