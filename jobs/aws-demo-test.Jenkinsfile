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
                    name: 'OWNER_NUMBER',
                    description: "AWS Owner number",
                    $class: 'hudson.model.StringParameterDefinition'
                ],
                [
                    name: 'AWS_REGION',
                    $class: 'hudson.model.StringParameterDefinition',
                    description: 'AWS region',
                    defaultValue: 'us-east-1'
                ],
                [
                    name: 'AWS_CREDENTIALS',
                    $class: 'hudson.model.StringParameterDefinition',
                    description: 'AWS access credentials',
                    defaultValue: 'kubevirt-demos'
                ],
                [
                    name: 'INSTANCE_KEYPAIR_NAME',
                    description: "AWS SSH Keypair Name",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'kubevirt-demos-ssh-name'
                ],
                [
                    name: 'INSTANCE_SSH_PRIVATE_KEY',
                    description: "Name of the SSH credentials for AWS instance",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'kubevirt-demos-ssh'
                ],
                [
                    name: "INSTANCE_SSH_USERNAME",
                    description: "The username for SSH to the instance",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'centos'
                ],
                [
                    name: 'DEMO_NAME',
                    description: "The username for SSH to the instance",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'lab1'
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
                    name: "NOTIFY_EMAIL_PASS",
                    description: "A comma separated list of email addressed to notify on success",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: ''  
                ],
                [
                    name: "NOTIFY_EMAIL_FAIL",
                    description: "A comma separated list of email addressed to notify on failure",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: ''  
                ],
                [
                    name: 'PERSIST',
                    description: 'leave the minikube service in place',
                    $class: 'hudson.model.BooleanParameterDefinition',
                    defaultValue: false
                ],
                [
                    name: 'DEBUG',
                    description: 'ask commands to print details',
                    $class: 'hudson.model.BooleanParameterDefinition',
                    defaultValue: false
                ]

            ]
        ]
    ]
)

persist = PERSIST.toBoolean()
debug = DEBUG.toBoolean()

def setupJob
def executeJob
def teardownJob

node(TARGET_NODE) {
    
    sh "aws configure set region ${AWS_REGION}"

    stage("create instance") {
        setupJob = build(
            job: 'aws-setup',
            propagate: true,
            parameters: [
                [
                    name: 'TARGET_NODE',
                    value: TARGET_NODE,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'OWNER_NUMBER',
                    value: OWNER_NUMBER,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'AWS_REGION',
                    value: AWS_REGION,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'AWS_CREDENTIALS',
                    value: AWS_CREDENTIALS,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'INSTANCE_KEYPAIR_NAME',
                    value: INSTANCE_KEYPAIR_NAME,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'INSTANCE_SSH_PRIVATE_KEY',
                    value: INSTANCE_SSH_PRIVATE_KEY,
                    $class: 'StringParameterValue'
                ],
                [
                    name: 'INSTANCE_SSH_USERNAME',
                    value: INSTANCE_SSH_USERNAME,
                    $class: 'StringParameterValue'
                ]
            ]
        )
        currentBuild.displayName = "kubevirt-demos:${setupJob.displayName}"
        currentBuild.result = setupJob.result

        // grab the returned INSTANCE_ID from the build job variables
        AWS_INSTANCE_ID = setupJob.getBuildVariables().INSTANCE_ID
        AWS_INSTANCE_DNS_NAME = setupJob.getBuildVariables().INSTANCE_PUBLIC_DNS_NAME
    }

    try {
        stage("execute demo") {
            executeJob = build(
                job: "demo-test",
                propagate: true,
                parameters: [
                    [
                        name: 'TARGET_NODE',
                        value: TARGET_NODE,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'INSTANCE_DNS_NAME',
                        value: AWS_INSTANCE_DNS_NAME,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'INSTANCE_SSH_PRIVATE_KEY',
                        value: INSTANCE_SSH_PRIVATE_KEY,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'INSTANCE_SSH_USERNAME',
                        value: INSTANCE_SSH_USERNAME,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'DEMO_NAME',
                        value: DEMO_NAME,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'DEMO_GIT_REPO',
                        value: DEMO_GIT_REPO,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'DEMO_GIT_BRANCH',
                        value: DEMO_GIT_BRANCH,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'DEMO_ROOT',
                        value: DEMO_ROOT,
                        $class: 'StringParameterValue'
                    ]
                ]
            )

            copyArtifacts(
                projectName: 'demo-test',
                selector: specific("${executeJob.number}")
            )
        }
    } catch (error) {
        // report demo failure
        if (NOTIFY_EMAIL_FAIL != '') {
            echo "Sending failure email to ${NOTIFY_EMAIL_FAIL}"
            // Compose the body of a FAIL email
            // Start time
            // End time
            // Duration
            // Stdout
            startTime = new Date(currentBuild.startTimeInMillis)
            demoStartTime = new Date(executeJob.startTimeInMillis)
            
            body = """
Name           : aws-demo-test ${currentBuild.number}
Start Time     : ${startTime.toString()}
Total Duration : ${currentBuild.durationString}
Total Status   : ${currentBuild.currentResult}
Total URL      : ${currentBuild.absoluteUrl}

Demo Name      : ${DEMO_NAME}
Demo Start Time: ${demoStartTime}
Demo Duration  : ${executeJob.durationString}
Demo Status    : ${executeJob.currentResult}
Demo URL       : ${executeJob.absoluteUrl}

"""

            mail(
                to: NOTIFY_EMAIL_FAIL,
                from: "kubevirt-demo-test@redhat.com",
                replyTo: "mlamouri+jenkins@redhat.com",
                subject: "[aws-demo-test] FAIL",
                body: body
            )
        }
    } finally {

        stage("teardown instance") {

            echo "AWS_INSTANCE_ID = ${AWS_INSTANCE_ID}"

            teardownJob = build(
                job: 'aws-teardown',
                propagate: true,
                parameters: [
                    [
                        name: 'TARGET_NODE',
                        value: TARGET_NODE,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'AWS_CREDENTIALS',
                        value: AWS_CREDENTIALS,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'AWS_REGION',
                        value: AWS_REGION,
                        $class: 'StringParameterValue'
                    ],
                    [
                        name: 'AWS_INSTANCE_ID',
                        value: AWS_INSTANCE_ID,
                        $class: 'StringParameterValue'
                    ]
                ]
            )
        }

        archiveArtifacts artifacts: "demo-test-result-*.txt"

        if (!persist) {
            cleanWs()
            deleteDir()
        } 
    }

}

currentBuild.result = executeJob.currentResult

if (executeJob.currentResult == 'SUCCESS' && NOTIFY_EMAIL_PASS != '') {
    echo "Sending success email to ${NOTIFY_EMAIL_PASS}"
    // Compose the body of a PASS email
    // Start time
    // End time
    // Duration
    // Stdout
    startTime = new Date(currentBuild.startTimeInMillis)
    demoStartTime = new Date(executeJob.startTimeInMillis)
    
    body = """
Name           : aws-demo-test ${currentBuild.number}
Start Time     : ${startTime.toString()}
Total Duration : ${currentBuild.durationString}
Total Status   : ${currentBuild.currentResult}
Total URL      : ${currentBuild.absoluteUrl}

Demo Name      : ${DEMO_NAME}
Demo Start Time: ${demoStartTime}
Demo Duration  : ${executeJob.durationString}
Demo Status    : ${executeJob.currentResult}
Demo URL       : ${executeJob.absoluteUrl}

"""

    mail(
        to: NOTIFY_EMAIL_PASS,
        from: "kubevirt-demo-test@redhat.com",
        replyTo: "mlamouri+jenkins@redhat.com",
        subject: "[aws-demo-test] PASS",
        body: body
    )
} else {
    echo "No recipients for PASS email provided"
}

