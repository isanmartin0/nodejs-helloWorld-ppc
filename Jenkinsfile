#!/usr/bin/groovy

def runNodejsPPCJenkinsfile() {


    echo "BEGIN NODE.JS PARALLEL CONFIGURATION PROJECT (PGC)"

    node('nodejs') {

        stage('Checkout') {
            echo 'Getting source code... parallel project pipeline'
            checkout scm
            projectURL = scm.userRemoteConfigs[0].url
            echo "Source code hosted in: ${projectURL}"
        }

    }

    echo "END NODE.JS GENERIC CONFIGURATION PROJECT (PGC)"

} //end of method

return this;

