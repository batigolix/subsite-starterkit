    wrap([$class: 'AnsiColorBuildWrapper', cxolorMapName: 'xterm']) {

        def buildId = sh(returnStdout: true, script: 'date |  md5sum | head -c 5').trim()
        def buildName = "${env.JOB_NAME}".replaceAll('%2F','_').replaceAll('/','_').replaceAll('-','_').trim()
        def buildLink = "<${env.BUILD_URL}consoleFull|${buildName} #${env.BUILD_NUMBER}>"
        //def releaseName = props['project.id'] + "_" + sh(returnStdout: true, script: 'date +%Y%m%d%H%M%S').trim() + "_${props['platform.package.reference']}"
        //def releasePath = "/usr/share/subsites/releases/${props['project.id']}"

        withEnv([
            "WORKSPACE=${env.WORKSPACE}",
            "WD_HOST_URL=http://127.0.0.1:8647/wd/hub",
            "BUILD_ID_UNIQUE=${buildName}_${buildId}",
        ]) {

            stage('Init') {
                deleteDir()
                checkout scm
                setBuildStatus("Build started.", "PENDING");
                slackSend color: "good", message: "Subsite build ${buildLink} started."
                //sh "docker run -u jenkins -v ${WORKSPACE}:/app -v /usr/share/composer:/usr/share/composer docker_composer install --no-suggest --no-interaction"
                //sh "./ssk/phing start-container -D'docker.container.id'=${buildId} -D'docker.container.workspace'=${WORKSPACE}"
                sh "mkdir -p ${WORKSPACE}/platform"
                sh "docker-compose -f resources/docker/docker-compose.yml up -d"
             }

            try {
                stage('Check') {
                    //dockerExecute('composer', 'update --no-suggest --no-interaction')
                    //dockerExecute('./ssk/phing', 'setup-php-codesniffer quality-assurance') 
                }


                stage('Build') {
                    dockerExecute('./ssk/phing', "build-dev -D'behat.wd_host.url'='http://selenium:4444/wd/hub' -D'behat.browser.name'='chrome'")
                }

                stage('Test') {
                    dockerExecute('./ssk/phing', "install-dev -D'drupal.db.host'='mysql' -D'drupal.db.name'='${env.BUILD_ID_UNIQUE}'")
                    timeout(time: 2, unit: 'HOURS') {
                        //dockerExecute('phantomjs', '--webdriver=127.0.0.1:8643 &')
                        dockerExecute('./ssk/behat', '-c tests/behat.yml --strict')
                    }
                }

                stage('Package') {
                    dockerExecute('./ssk/phing', "build-release -D'project.release.path'='${releasePath}' -D'project.release.name'='${releaseName}'")
                    setBuildStatus("Build complete.", "SUCCESS");
                    slackSend color: "good", message: "Subsite build ${buildLink} completed."
                }
            } catch(err) {
                setBuildStatus("Build failed.", "FAILURE");
                slackSend color: "danger", message: "Subsite build ${buildLink} failed."
                throw(err)
            } finally {
                sh "docker-compose -f resources/docker/docker-compose.yml down"
            }
        }
    }
}

void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
//        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "${env.BUILD_CONTEXT}"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]]]
    ]);
}

def dockerExecute(String executable, String command) {
    switch("${executable}") {
        case "./ssk/phing":
            color = "-logger phing.listener.AnsiColorLogger"
            break
        case "./ssk/behat":
            color = "--colors"
            break
        case "composer":
            color = "--ansi"
            break
        default:
            color = ""
            break
    }
    sh "docker exec -u web ${BUILD_ID_UNIQUE}_php ${executable} ${command} ${color}"
