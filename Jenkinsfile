//
// Pipeline con dos repos
//
println "** Pipeline script embedded"
println "** BUILD con BRANCH parametro"
// ---------------------------Agents labels
def linuxAgent = 'master'
def zosAgent   = 'zPDT'
// ---------------------------Verbose
def buildVerbose = 'false'
// ---------------------------Hosts and ports
def dbbWebApp = '192.168.1.37'
def zosHost = '192.168.1.200'
def zosPort = '22'
// ----------------------------DBB https
def dbbUrl = 'https://'+dbbWebApp+':9443/dbb'  
def dbbHlq = 'MVS.DBB'
// ---------------------------- Git (GitHub)
def gitCred    = 'cfdonatucci'
def srcGitBranch = '${BRANCH}'
def srcGitRepo1 = 'git@github.com:cfdonatucci/testApp.git'
def srcGitRepo2 = 'git@github.com:cfdonatucci/shrInt.git'
// ----------------------------- Build type
//  -i: incremental
//  -f: full
def buildType='-f'
// ----------------------------  Build extra args
//  -d: COBOL debug options
def buildExtraParams='-d'
// ===========================================================
pipeline { 
  agent { label zosAgent }
      environment { WORK_DIR = "${WORKSPACE}/builds/build-${BUILD_NUMBER}" }
      options { skipDefaultCheckout(true) }
// -------------------------------------------------------------------------
  stages { 
    stage('Init') {
     steps {
      script {
        env.DBB_HOME = '/var/dbb'
        echo "WorkSpace : ${WORKSPACE}/builds/build-${BUILD_NUMBER}"
        echo "Repository: ${srcGitRepo1} - branch: ${srcGitBranch} "
        echo "Repository: ${srcGitRepo2} - branch: ${srcGitBranch} "

        }
       }
      }
// -------------------------------------------------------------------------
    stage('Git Clone/Refresh') {
        agent { label zosAgent }
        steps {
            sh(script: 'rm -rf ${WORKSPACE}/testApp', returnStdout: true)
            sh(script: 'rm -rf ${WORKSPACE}/testApp@tmp', returnStdout: true)
            sh(script: 'rm -rf ${WORKSPACE}/shrInt', returnStdout: true)
            sh(script: 'rm -rf ${WORKSPACE}/shrInt@tmp', returnStdout: true)
            script {
                println "** >branch: ${srcGitBranch}"
                println "** >WORKSPACE  ${WORKSPACE}"

                dir('testApp') {
                checkout([$class: 'GitSCM', branches: [[name: srcGitBranch]], doGenerateSubmoduleConfigurations: false,
                  submoduleCfg: [], userRemoteConfigs: [[credentialsId: gitCred,url: srcGitRepo1 ]]])}

                dir('shrInt') {
                checkout([$class: 'GitSCM', branches: [[name: srcGitBranch]], doGenerateSubmoduleConfigurations: false,
                   submoduleCfg: [], userRemoteConfigs: [[credentialsId: gitCred,url: srcGitRepo2 ]]])}
              
               } } }
// -----------------------------------ws /u/carlos/jenkins --------------------------------------
    stage('DBB Build') {
      steps {
        script{
          node( zosAgent ) {
            
sh "$DBB_HOME/bin/groovyz /u/adcdmst/zAppBuild/build.groovy $buildType -w ${WORKSPACE} -o ${WORKSPACE} -a testApp -h ${dbbHlq}.STG "

  
          }
        }
      }
    }
  }//pipeline
}//stages