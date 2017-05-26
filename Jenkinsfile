#!/usr/bin/env groovy

properties([
    buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5')),
    pipelineTriggers([cron('@daily')]),
])

node('docker') {
    deleteDir()

    stage('Checkout') {
        checkout scm
    }

    if (!infra.isTrusted()) {
        /* Outside of the trusted.ci environment, we're building and testing
         * the Dockerful in this repository, but not publishing to docker hub
         */
        stage('Build') {
            docker.build('jenkins')
        }

        stage('shellcheck') {
            docker.image('koalaman/shellcheck').inside() {
                // run shellcheck ignoring error SC1091
                // Not following: /usr/local/bin/jenkins-support was not specified as input
                sh """
                find / -iname install-plugins.sh
                shellcheck -e SC1091 *.sh
                """
            }
        }

        stage('Test') {
            sh """
            git submodule update --init --recursive
            git clone https://github.com/sstephenson/bats.git
            bats/bin/bats tests
            """
        }
    } else {
        /* In our trusted.ci environment we only want to be publishing our
         * containers from artifacts
         */
        stage('Publish') {
            sh './publish.sh'
        }
    }
}
