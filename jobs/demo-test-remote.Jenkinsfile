#!/usr/bin/env groovy

properties(
    [
        disableConcurrentBuilds(),
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '',
                numToKeepStr: '5')),
        [
            $class: 'ParametersDefinitionProperty',
            parameterDefinitions: [
                [
                    name: 'TARGET_NODE',
                    description: 'Jenkins agent node',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'awscli'
                ],
                [
                    name: "DEMO_NAME",
                    description: "The name of the demo to run",
                    $class: 'hudson.model.StringParameterDefinition',
                ],
                [
                    name: "DEMO_GIT_REPO",
                    description: "Where to find the demo page and test code",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "https://github.com/kubevirt/kubevirt.github.io.git"
                ],
                [
                    name: "DEMO_GIT_BRANCH",
                    description: "The branch that contains the of the demo to run",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "master"
                ],
                [
                    name: "DEMO_ROOT",
                    description: "The directory that contains the demo tests",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: '_includes/scriptlets'
                ],
                [
                    name: 'INSTANCE_DNS_NAME',
                    description: "Name of host to execute the demo",
                    $class: 'hudson.model.StringParameterDefinition',
                ],
                [
                    name: 'INSTANCE_SSH_PRIVATE_KEY',
                    description: "Name of the SSH credentials for instance",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'kubevirt-demos-ssh'
                ],
                [
                    name: "INSTANCE_SSH_USERNAME",
                    description: "The username for SSH to the instance",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'centos'
                ]
            ]
        ]
    ]
)

SSH_HOST_SPEC = "${INSTANCE_SSH_USERNAME}@${INSTANCE_DNS_NAME}"
SSH_OPTIONS = "-o StrictHostKeyChecking=no"
SSH = "ssh ${SSH_OPTIONS} ${SSH_HOST_SPEC}"
SCP = "scp ${SSH_OPTIONS}"

node(TARGET_NODE) {

    checkout scm
    
    sshagent([INSTANCE_SSH_PRIVATE_KEY]) {

        stage('push demo test script') {
            echo "pushing demo test script"
            sh "${SSH} mkdir -p bin demos"
            sh "${SCP} scripts/run_demo.py ${SSH_HOST_SPEC}:bin"
            sh "${SSH} chmod a+x bin/\\*"
        }

        stage('configure kubectl') {
            echo "configure kubectl on remote"
            // return the status to avoid error on no-create
            cmd_status = sh(
                returnStatus: true,
                script: "${SSH} 'mkdir -p ~/.kube ; [ -r ./admin.conf -a ! -r .kube/config ] && ln -s ~/admin.conf .kube/config'"
            )

            // sh "${SCP} localfile ${SSH_HOST_SPEC}:dest"
            // sh "${SSH} chmod a+x bin/\*"
        }

        stage('clone demo repo') {
            echo "cloning demo repo"
            checkout(
                [
                    $class: "GitSCM",
                    userRemoteConfigs: [
                        [
                            url: DEMO_GIT_REPO
                        ]
                    ],
                    branches: [
                        [
                            name: DEMO_GIT_BRANCH
                        ]
                    ],
                    extensions: [
                        [
                            $class: 'RelativeTargetDirectory',
                            relativeTargetDir: 'demos'
                        ]
                    ]
                ]
            )
        }
        
        stage('push demo files') {
            echo "push demo files"
            dir('demos') {
                sh "${SCP} -r ${DEMO_ROOT}/${DEMO_NAME} ${SSH_HOST_SPEC}:demos/"
            }
        }

        stage("execute test") {
            echo "execute test"
            result = sh (
                returnStdout: true,
                script: "${SSH} bin/run_demo.py -t demos/${DEMO_NAME}"
            )
            echo "result = --- \n${result}\n---"
            
            writeFile(
                file: "demo-test-result-${demo_name}.txt",
                text: "${result}"
            )
        }
    }


    archiveArtifacts artifacts: "demo-test-result-*.txt"
}
