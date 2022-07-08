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
                PROXY_PWD = credentials('apn-proxy')
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'openshift-nexus',
                                          passwordVariable: 'NXPASS',
                                          usernameVariable: 'NXUSER')]) {
                    sh """
                         echo " {\"proxies\":{\"default\": { \"httpProxy\": "http://${PROXY_PWD}@proxy.meteoswiss.ch:8080",
                                                            \"httpsProxy\": "http://${PROXY_PWD}@proxy.meteoswiss.ch:8080" }}} " > ~/.docker/config.json
                        docker pull registry.hub.docker.com/library/ubuntu:20.04

                        echo \$NXPASS | docker login docker-all-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        echo \$NXPASS | docker login docker-intern-nexus.meteoswiss.ch -u \$NXUSER --password-stdin
                        export COMPOSE_DOCKER_CLI_BUILD=1
                        export DOCKER_BUILDKIT=1
                        echo "proxy $http_proxy"
                        cp -r /etc/ssl/certs .
                        ls -ltr ~/.docker/
                        cat ~/.docker/config.json
                        echo $http_proxy
                        export http_proxy=http://${PROXY_PWD}@proxy.meteoswiss.ch:8080
                        export https_proxy=https://${PROXY_PWD}@proxy.meteoswiss.ch:8080
                        export HTTP_PROXY=http_proxy
                        export HTTPS_PROXY=https_proxy
                        export no_proxy=localhost,.meteoswiss.ch,ssi.dwd.de

                        docker pull registry.hub.docker.com/library/ubuntu:20.04
                        docker build --build-arg PROXY_PWD=${PROXY_PWD} .
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
