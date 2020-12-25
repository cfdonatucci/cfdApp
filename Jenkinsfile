//
// Pipeline con dos repos y sparce checkout
//
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
//def srcGitBranch = '${BRANCH}'
//def srcGitBranch = 'main'
def srcGitRepo1 = 'git@github.com:cfdonatucci/cfdApp.git'
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
      parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch Name')
      }

      environment { 
        WORK_DIR = "${WORKSPACE}/builds/build-${BUILD_NUMBER}"
        DBB_HOME = '/var/dbb'
      }
      options { skipDefaultCheckout(true) }
// -------------------------------------------------------------------------
  stages { 
    stage('Init') {
     steps {
      script {
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
            script {
              def srcGitBranch = ${params.BRANCH}

                dir('testApp') {
                checkout([$class: 'GitSCM', branches: [[name: srcGitBranch]], doGenerateSubmoduleConfigurations: false,
                  submoduleCfg: [], userRemoteConfigs: [[credentialsId: gitCred,url: srcGitRepo1, ]]])}

                dir ('shrInt') {
                sh(script: 'rm -f .git/info/sparse-checkout', returnStdout: true)
                def scmVars = checkout([$class: 'GitSCM', branches: [[name: srcGitBranch]], doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'SparseCheckoutPaths', sparseCheckoutPaths:[[$class:'SparseCheckoutPath', path:'copybook/*']]]],
                    submoduleCfg: [], userRemoteConfigs: [[ credentialsId: gitCred,url: srcGitRepo2, ]]]) }
              
               } } }
// -----------------------------------ws /u/carlos/jenkins --------------------------------------
    stage('DBB Build') {
      steps {
        script{
          node( zosAgent ) {
            
sh "$DBB_HOME/bin/groovyz /u/adcdmst/zAppBuild/build.groovy $buildType -w ${WORKSPACE} -o ${WORK_DIR} -a cfdApp -h ${dbbHlq}.STG "
  
          }//node
        }//script
      }//steps

      post {
          always {
              node( zosAgent ) {
               sh(script: 'rm -rf ${WORKSPACE}/cfdApp*', returnStdout: true)
               sh(script: 'rm -rf ${WORKSPACE}/shrInt*', returnStdout: true)
               }
             }
          }//post

    }//stage

  }//stages
}//pipeline
