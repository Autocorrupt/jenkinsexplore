#!/usr/bin/env groovy

properties([buildDiscarder(daysToKeepStr: '30')])

def isMasterBranch() {
  env.BRANCH_NAME == 'master'
}

/*
def awsCredentials = [[$class: 'AmazonWebServicesCredentialsBinding',
  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
  credentialsId: 'docker-qa@docker-core.aws',
  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
  file(credentialsId: 'docker-qa.docker-core.pem', variable: 'AWS_KEY_PATH')]
*/

def testStepName(config) {
  "TEST distro ${config.distro}" 
}

def testStepBody(config) {
  { ->
    sh("echo make DISTRO=${config.distro} TESTNAME=E2E-CI-${BRANCH_NAME}-BUILD-${BUILD_ID} AWS_KEY_NAME=docker-qa ci")
    sh("env")
    if (config.distro == 'centos')
      sh("blah")
  }
}

def gitLogStr

// Returns a [name:closure] entry suitable for parallel()
def testStep(config) {
  return [(testStepName(config)) : (testStepBody(config))]
}

// Test configs to run for PR builds
def matrix = [[distro:'ubuntu'], [distro:'centos']]

// Aditional test configs for merges to master branch
if (isMasterBranch()) {
  matrix += [[distro:'centos']]
  // Just examples to illustrate use case
  // matrix << [[distro:'centos'  ],
  //            [distro:'debian'  ],
  //            [distro:'sles'    ],
  //            [distro:'linuxkit'],
  //            [distro:'rhel'    ]]
}

def steps = [:]
for (config in matrix) {
  steps << testStep(config)
}

try {
    stage('build') {
        // TODO: checkout scm, build and push e2e docker image in a wrappedNode
    }

    stage('test') {
        node() {
            // TODO: move checkout to 'build' stag
            checkout scm
            gitLogStr = sh(returnStdout: true, script: 'git show -s --pretty=format:"%h: %an: %s" HEAD')
            parallel(steps) 
        }
    }
} catch(err) {
   echo("SEND SLACK HERE")
   echo("Error: ${err}")
   echo("Git log: ${gitLogStr}")
   throw err
}

