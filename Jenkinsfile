def extcode

node {
 wrap([$class: 'AnsiColorBuildWrapper', cxolorMapName: 'xterm']) {
   deleteDir()
   checkout scm
   sh "composer update --no-interaction --no-suggest"
   extcode = load "vendor/ec-europa/ssk/Jenkinsfile"
   extcode.createWorkflow()
 }
}
