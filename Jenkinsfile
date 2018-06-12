#!/usr/bin/groovy
import com.evobanco.NodejsUtils
import com.evobanco.NodejsConstants

def runNodejsPPCJenkinsfile() {


    def utils = new NodejsUtils()

    def npmRepositoryURL = 'http://10.6.14.20:8081/artifactory/api/npm/npm-repo/'
    def npmLocalRepositoryURL = 'http://10.6.14.20:8081/artifactory/api/npm/npm-local/'

    def openshiftURL = 'https://openshift.grupoevo.corp:8443'
    def openshiftCredential = 'openshift'
    def registry = '172.20.253.34'
    def artifactoryCredential = 'artifactory-token'
    def artifactoryNPMAuthCredential = 'artifactory-npm-auth'
    def artifactoryNPMEmailAuthCredential = 'artifactory-npm-email-auth'
    def jenkinsNamespace = 'cicd'
    def params
    String envLabel
    String branchName
    String branchNameHY
    String branchType


    //Parallel project configuration (PPC) properties
    def branchPPC = 'master'
    String credentialsIdPPCDefault = '4b18ea85-c50b-40f4-9a81-e89e44e20178' //credentials of the parallel configuration project
    def credentialsIdPPC
    def relativeTargetDirPPC = '/tmp/configs/PPC/'
    def isPPCJenkinsFile = false
    def isPPCJenkinsYaml = false
    def isPPCOpenshiftTemplate = false
    def jenkinsFilePathPPC = relativeTargetDirPPC + 'Jenkinsfile'
    def jenkinsYamlPathPPC = relativeTargetDirPPC + 'Jenkins.yml'
    def openshiftNodejsTemplatePathPPC = relativeTargetDirPPC + 'kube/nodejs_template.yaml'
    def jenknsFilePipelinePPC


    //Generic project configuration properties
    def gitDefaultProjectConfigurationPath='https://github.com/isanmartin0/evo-cicd-nodejs-generic-configuration'
    def relativeTargetDirGenericPGC = '/tmp/configs/generic/'
    def branchGenericPGC = 'master'
    def credentialsIdGenericPGC = '4b18ea85-c50b-40f4-9a81-e89e44e20178' //credentials of the generic configuration project
    def jenkinsYamlGenericPath = relativeTargetDirGenericPGC + 'Jenkins.yml'
    def openshiftNodejsTemplateGenericPath = relativeTargetDirGenericPGC + 'kube/nodejs_template.yaml'


    def packageJSON
    String projectURL
    String packageName
    String packageVersion
    String packageTag
    String packageTarball
    String packageViewTarball
    boolean isScopedPackage = false
    String packageScope


    int maxOldBuildsToKeep = 0
    int daysOldBuildsToKeep = 0

    //Taurus parameters
    def taurus_test_base_path = 'taurus'
    def acceptance_test_path = '/acceptance_test/'
    def performance_test_path = '/performance_test/'
    def smoke_test_path = '/smoke_test/'
    def security_test_path = '/security_test/'


    def openshift_route_hostname = ''
    def openshift_route_hostname_with_protocol = ''

    //Parameters nodejs
    int port_default = 8080
    int debug_port_default = 5858
    int image_stream_nodejs_version_default = 8

    def build_from_registry_url = 'https://github.com/isanmartin0/s2i-nodejs-container.git'
    def build_from_artifact_branch = 'master'

    def nodeJS_8_installation = "Node-8.9.4"
    def nodeJS_6_installation = "Node-6.11.3"
    def nodeJS_pipeline_installation = ""
    int image_stream_nodejs_version = image_stream_nodejs_version_default
    def sonarProjectPath = "sonar-project.properties"

    echo "BEGIN NODE.JS PARALLEL CONFIGURATION PROJECT (PPC)"

    node('nodejs') {

        stage('Checkout') {
            echo 'Getting source code... parallel project pipeline'
            checkout scm
            projectURL = scm.userRemoteConfigs[0].url
            echo "Source code hosted in: ${projectURL}"
        }

        //sleep 10
        checkout scm

        stage('Check Parallel project configuration elements (PPC)') {

            packageJSON = readJSON file: 'package.json'

            isScopedPackage = utils.isScopedPackage(packageJSON.name)
            echo "isScopedPackage: ${isScopedPackage}"

            if (isScopedPackage) {
                packageScope = utils.getPackageScope(packageJSON.name)
                echo "packageScope: ${packageScope}"
            }

            packageName = utils.getProject(packageJSON.name)
            echo "packageName: ${packageName}"
            packageVersion = packageJSON.version
            echo "packageVersion: ${packageVersion}"
            packageTag = utils.getPackageTag(packageJSON.name, packageVersion)
            echo "packageTag: ${packageTag}"
            packageTarball = utils.getPackageTarball(packageJSON.name, packageVersion)
            echo "packageTarball: ${packageTarball}"
            packageViewTarball = utils.getPackageViewTarball(packageJSON.name, packageVersion)
            echo "packageViewTarball: ${packageViewTarball}"



            try {
                def parallelConfigurationProject = utils.getParallelConfigurationProjectURL(projectURL)

                echo "Node.js parallel configuration project ${parallelConfigurationProject} searching"

                retry (3)
                        {
                            checkout([$class                           : 'GitSCM',
                                      branches                         : [[name: branchPPC]],
                                      doGenerateSubmoduleConfigurations: false,
                                      extensions                       : [[$class           : 'RelativeTargetDirectory',
                                                                           relativeTargetDir: relativeTargetDirPPC]],
                                      submoduleCfg                     : [],
                                      userRemoteConfigs                : [[credentialsId: credentialsIdPPC,
                                                                           url          : parallelConfigurationProject]]])
                        }

                echo "Node.js Parallel configuration project ${parallelConfigurationProject} exits"

                // Jenkinsfile
                isPPCJenkinsFile = fileExists jenkinsFilePathPPC

                if (isPPCJenkinsFile) {
                    echo "Node.js Parallel configuration project Jenkinsfile... FOUND"
                } else {
                    echo "Node.js Parallel configuration project Jenkinsfile... NOT FOUND"
                }


                // Jenkins.yml
                isPPCJenkinsYaml = fileExists jenkinsYamlPathPPC

                if (isPPCJenkinsYaml) {
                    echo "Node.js Parallel configuration project Jenkins.yml... FOUND"
                } else {
                    echo "Node.js Parallel configuration project Jenkins.yml... NOT FOUND"
                }

                // Openshift template (template.yaml)
                isPPCOpenshiftTemplate = fileExists openshiftNodejsTemplatePathPPC

                if (isPPCOpenshiftTemplate) {
                    echo "Node.js Parallel configuration project Openshift template... FOUND"
                } else {
                    echo "Node.js Parallel configuration project Openshift template... NOT FOUND"
                }


                echo "isPPCJenkinsFile : ${isPPCJenkinsFile}"
                echo "isPPCJenkinsYaml : ${isPPCJenkinsYaml}"
                echo "isPPCOpenshiftTemplate : ${isPPCOpenshiftTemplate}"

            }
            catch (exc) {
                echo 'There is an error on retrieving Node.js parallel project configuration'
                def exc_message = exc.message
                echo "${exc_message}"
            }
        }

       stage('Load pipeline configuration') {


            if (!isPPCJenkinsFile || !isPPCJenkinsYaml || !isPPCOpenshiftTemplate) {
                currentBuild.result = NodejsConstants.FAILURE_BUILD_RESULT
                throw new hudson.AbortException('The parallel project configuration has not mandatory elements')
            }

            //Take parameters of the parallel project configuration (PPC)
            params = readYaml  file: jenkinsYamlPathPPC
            echo "Using Jenkins.yml from parallel project configuration (PPC)"

            //The template is provided by parallel project configuration (PPC)
            params.openshift.templatePath = relativeTargetDirPPC + params.openshift.templatePath
            echo "Template provided by parallel project configuration (PPC)"

            assert params.openshift.templatePath?.trim()

            echo "params.openshift.templatePath: ${params.openshift.templatePath}"

         }

        stage('NodeJS initialization') {
            echo 'Node initializing...'

            /*************************************************************
             ************* IMAGE STREAM TAG NODE VERSION *****************
             *************************************************************/
            echo "params.imageStreamNodejsVersion: ${params.imageStreamNodejsVersion}"

            String imageStreamNodejsVersionParam = params.imageStreamNodejsVersion
            if (imageStreamNodejsVersionParam != null && imageStreamNodejsVersionParam.isInteger()) {
                image_stream_nodejs_version = imageStreamNodejsVersionParam as Integer
            }

            if (image_stream_nodejs_version >= 8) {
                echo "Assigning NodeJS installation ${nodeJS_8_installation}"
                nodeJS_pipeline_installation = nodeJS_8_installation
            } else if (image_stream_nodejs_version >= 6) {
                echo "Assigning NodeJS installation ${nodeJS_6_installation}"
                nodeJS_pipeline_installation = nodeJS_6_installation
            } else {
                currentBuild.result = "FAILED"
                throw new hudson.AbortException("Error checking existence of package on NPM registry")
            }

            def node = tool name: "${nodeJS_pipeline_installation}", type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
            env.PATH = "${node}/bin:${env.PATH}"

            echo 'Node version:'
            sh "node -v"

            echo 'NPM version:'
            sh "npm -v"
        }


        stage('Prepare') {
            echo "Prepare stage (PGC)"

            nodejsSetDisplayName()

            echo "${currentBuild.displayName}"

            branchName = utils.getBranch()
            echo "We are on branch ${branchName}"
            branchType = utils.getBranchType(branchName)
            echo "This branch is a ${branchType} branch"
            branchNameHY = branchName.replace("/", "-").replace(".", "-").replace("_","-")
            echo "Branch name processed: ${branchName}"

        }

        stage ('Openshift environment') {
            switch (branchType) {
                case 'feature':
                    echo "Detect feature type branch"
                    envLabel="dev"
                    break
                case 'develop':
                    echo "Detect develop type branch"
                    envLabel="dev"
                    break
                case 'release':
                    echo "Detect release type branch"
                    envLabel="uat"
                    break
                case 'master':
                    echo "Detect master type branch"
                    envLabel="pro"
                    break
                case 'hotfix':
                    echo "Detect hotfix type branch"
                    envLabel="uat"
                    break
            }
            echo "Environment selected: ${envLabel}"
        }


        withCredentials([string(credentialsId: "${artifactoryNPMAuthCredential}", variable: 'ARTIFACTORY_NPM_AUTH'), string(credentialsId: "${artifactoryNPMEmailAuthCredential}", variable: 'ARTIFACTORY_NPM_EMAIL_AUTH')]) {
            withEnv(["NPM_AUTH=${ARTIFACTORY_NPM_AUTH}", "NPM_AUTH_EMAIL=${ARTIFACTORY_NPM_EMAIL_AUTH}"]) {
                withNPM(npmrcConfig: 'my-custom-npmrc') {

                    if (branchName != 'master') {

                        stage('Build') {
                            echo 'Building dependencies...'
                            sh 'npm i'
                        }


                    }
                }
            }
        }

    }

    echo "END NODE.JS PARALLEL CONFIGURATION PROJECT (PPC)"

} //end of method

return this;

