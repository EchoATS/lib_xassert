@Library('xmos_jenkins_shared_library@develop') _

getApproval()

pipeline {
  agent none
  environment {
    REPO = 'lib_xassert'
  }
  options {
    buildDiscarder(xmosDiscardBuildSettings())
    skipDefaultCheckout()
    timestamps()
  }
  parameters {
    string(
      name: 'TOOLS_VERSION',
      defaultValue: '15.3.0',
      description: 'The XTC tools version'
    )
  }
  stages {
    stage('Build') {
      parallel {
        stage('xcore app build and run tests') {
          agent {
            label 'x86_64 && linux'
          }
          steps {
            dir("${REPO}") {
              checkout scm
 
              runLibraryChecks("${WORKSPACE}/${REPO}", "v2.0.0")

              dir("examples") {
                withTools(params.TOOLS_VERSION) {
                  sh 'cmake -G "Unix Makefiles" -B build'
                  sh 'xmake -C build -j 8'
                }
              }

              createVenv("requirements.txt")
              withVenv() {
                sh "pip install -r requirements.txt"
              }
              withTools(params.TOOLS_VERSION) {
                withVenv() {
                  dir("tests") {
                    sh 'cmake -G "Unix Makefiles" -B build'
                    sh 'xmake -C build -j 8'
                    sh 'pytest runtests.py'
                  }
                }
              }
            }
          }
          post {
            cleanup {
              xcoreCleanSandbox()
            }
          }
        }
        stage('Build docs') {
          agent {
            label 'documentation'
          }
          steps {
            dir("${REPO}") {
              checkout scm

              withXdoc("v2.0.20.2.post0") {
                withTools(params.TOOLS_VERSION) {
                  dir("${REPO}/doc") {
                    sh "xdoc xmospdf"
                    archiveArtifacts artifacts: "pdf/*.pdf"
                  }
                }
              }
            }
          }
          post {
            cleanup {
              xcoreCleanSandbox()
            }
          }
        }
      }
    }
  }
}
