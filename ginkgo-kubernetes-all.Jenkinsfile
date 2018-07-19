@Library('cilium') _

pipeline {
    agent {
        label 'baremetal'
    }

    parameters {
        string(defaultValue: '${ghprbPullDescription}', name: 'ghprbPullDescription')
        string(defaultValue: '${ghprbActualCommit}', name: 'ghprbActualCommit')
        string(defaultValue: '${ghprbTriggerAuthorLoginMention}', name: 'ghprbTriggerAuthorLoginMention')
        string(defaultValue: '${ghprbPullAuthorLoginMention}', name: 'ghprbPullAuthorLoginMention')
        string(defaultValue: '${ghprbGhRepository}', name: 'ghprbGhRepository')
        string(defaultValue: '${ghprbPullLongDescription}', name: 'ghprbPullLongDescription')
        string(defaultValue: '${ghprbCredentialsId}', name: 'ghprbCredentialsId')
        string(defaultValue: '${ghprbTriggerAuthorLogin}', name: 'ghprbTriggerAuthorLogin')
        string(defaultValue: '${ghprbPullAuthorLogin}', name: 'ghprbPullAuthorLogin')
        string(defaultValue: '${ghprbTriggerAuthor}', name: 'ghprbTriggerAuthor')
        string(defaultValue: '${ghprbCommentBody}', name: 'ghprbCommentBody')
        string(defaultValue: '${ghprbPullTitle}', name: 'ghprbPullTitle')
        string(defaultValue: '${ghprbPullLink}', name: 'ghprbPullLink')
        string(defaultValue: '${ghprbAuthorRepoGitUrl}', name: 'ghprbAuthorRepoGitUrl')
        string(defaultValue: '${ghprbTargetBranch}', name: 'ghprbTargetBranch')
        string(defaultValue: '${ghprbPullId}', name: 'ghprbPullId')
        string(defaultValue: '${ghprbActualCommitAuthor}', name: 'ghprbActualCommitAuthor')
        string(defaultValue: '${ghprbActualCommitAuthorEmail}', name: 'ghprbActualCommitAuthorEmail')
        string(defaultValue: '${ghprbTriggerAuthorEmail}', name: 'ghprbTriggerAuthorEmail')
        string(defaultValue: '${GIT_BRANCH}', name: 'GIT_BRANCH')
        string(defaultValue: '${ghprbPullAuthorEmail}', name: 'ghprbPullAuthorEmail')
        string(defaultValue: '${sha1}', name: 'sha1')
        string(defaultValue: '${ghprbSourceBranch}', name: 'ghprbSourceBranch')
    }

    environment {
        PROJ_PATH = "src/github.com/cilium/cilium"
        MEMORY = "3072"
        TESTDIR="${WORKSPACE}/${PROJ_PATH}/test"
        GOPATH="${WORKSPACE}"
    }

    options {
        timeout(time: 140, unit: 'MINUTES')
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'env'
                sh 'rm -rf src; mkdir -p src/github.com/cilium'
                sh 'ln -s $WORKSPACE src/github.com/cilium/cilium'
                checkout scm
            }
        }
        stage('Boot VMs'){
            steps {
                sh 'cd ${TESTDIR}; K8S_VERSION=1.8 vagrant up --no-provision'
                sh 'cd ${TESTDIR}; K8S_VERSION=1.9 vagrant up --no-provision'
            }
        }
        stage('BDD-Test-k8s') {
            environment {
                FAILFAST=setIfPR("true", "false")
            }
            options {
                timeout(time: 120, unit: 'MINUTES')
            }
            steps {
                parallel(
                    "K8s-1.8":{
                        sh 'cd ${TESTDIR}; K8S_VERSION=1.8 ginkgo --focus=" K8s*" -v --failFast=${FAILFAST}'
                    },
                    "K8s-1.9":{
                        sh 'cd ${TESTDIR}; K8S_VERSION=1.9 ginkgo --focus=" K8s*" -v --failFast=${FAILFAST}'
                    },
                )
            }
            post {
                always {
                    sh 'cd test/; ./archive_test_results.sh || true'
                    archiveArtifacts artifacts: "test_results_${JOB_BASE_NAME}_${BUILD_NUMBER}.zip", allowEmptyArchive: true
                    junit 'test/*.xml'
                    sh 'cd test/; ./post_build_agent.sh || true'
                }
            }
        }
        stage('Boot VMs k8s-next'){
            environment {
                GOPATH="${WORKSPACE}"
                TESTDIR="${WORKSPACE}/${PROJ_PATH}/test"
            }
            steps {
                sh 'cd ${TESTDIR}; K8S_VERSION=1.11 vagrant up --no-provision'
                sh 'cd ${TESTDIR}; K8S_VERSION=1.12 vagrant up --no-provision'
            }
        }
        stage('Non-release-k8s-versions') {
            environment {
                FAILFAST=setIfPR("true", "false")
            }
            options {
                timeout(time: 120, unit: 'MINUTES')
            }
            steps {
                parallel(
                    "K8s-1.11":{
                        sh 'cd ${TESTDIR}; K8S_VERSION=1.11 ginkgo --focus=" K8s*" --failFast=${FAILFAST}'
                    },
                    "K8s-1.12":{
                        sh 'cd ${TESTDIR}; K8S_VERSION=1.12 ginkgo --focus=" K8s*" --failFast=${FAILFAST}'
                    },
                )
            }
            post {
                always {
                    junit 'test/*.xml'
                    sh 'cd test/; ./post_build_agent.sh || true'
                    sh 'cd test/; ./archive_test_results.sh || true'
                    archiveArtifacts artifacts: "test_results_${JOB_BASE_NAME}_${BUILD_NUMBER}.zip", allowEmptyArchive: true
                }
            }
        }
    }
    post {
        always {
            sh "cd ${TESTDIR}; K8S_VERSION=1.8 vagrant destroy -f || true"
            sh "cd ${TESTDIR}; K8S_VERSION=1.9 vagrant destroy -f || true"
            sh "cd ${TESTDIR}; K8S_VERSION=1.10 vagrant destroy -f || true"
            sh "cd ${TESTDIR}; K8S_VERSION=1.11 vagrant destroy -f || true"
            sh "cd ${TESTDIR}; K8S_VERSION=1.12 vagrant destroy -f || true"
            sh "cd ${TESTDIR}; ./post_build_agent.sh || true"
            cleanWs()
        }
    }
}

