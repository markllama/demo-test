properties(
    [
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '',
                numToKeepStr: '360')
        ),
        disableConcurrentBuilds(),
        [
            $class: 'ParametersDefinitionProperty',
            parameterDefinitions: [
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
                    defaultValue: 'lab2'
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
                    defaultValue: DEFAULT_NOTIFY_EMAIL_PASS
                ],
                [
                    name: "NOTIFY_EMAIL_FAIL",
                    description: "A comma separated list of email addressed to notify on failure",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: DEFAULT_NOTIFY_EMAIL_FAIL
                ]
            ]
        ]
    ]
)

//
// OWNER_NUMBER as  ENVVAR
// AWS_REGION       ENVVAR
// AWS_CREDENTIALS       ENVVAR
// 
//
// DEMO_NAME
// DEMO_GIT_REPO
// DEMO_GIT_BRANCH

// NOTIFY_EMAIL_PASS
// NOTIFY_EMAIL_FAIL
//

demo = build(
    job: "kubevirt/aws-demo-test",
    propagate: false,
    parameters: [
        [
            name: 'OWNER_NUMBER',
            value: AWS_OWNER_NUMBER,
            $class: 'StringParameterValue'
        ],
        [
            name: 'AWS_CREDENTIALS',
            value: 'aws-credentials',
            $class: 'StringParameterValue'
        ],
        [
            name: 'INSTANCE_KEYPAIR_NAME',
            value: 'kubevirt-demos',
            $class: 'StringParameterValue'
        ],
        [
            name: 'INSTANCE_SSH_PRIVATE_KEY',
            value: 'kubevirt-demos',
            $class: 'StringParameterValue'
        ],
        [
            name: "DEMO_NAME",
            value: DEMO_NAME,
            $class: 'StringParameterValue'
        ],
        [
            name: "DEMO_GIT_REPO",
            value: DEMO_GIT_REPO,
            $class: 'StringParameterValue'
        ],
        [
            name: "DEMO_GIT_BRANCH",
            value: DEMO_GIT_BRANCH,
            $class: 'StringParameterValue'
        ],
        [
            name: "DEMO_ROOT",
            value: DEMO_ROOT,
            $class: 'StringParameterValue'
        ],
        [
            name: "NOTIFY_EMAIL_PASS",
            value: NOTIFY_EMAIL_PASS,
            $class: 'StringParameterValue'
        ],
        [
            name: "NOTIFY_EMAIL_FAIL",
            value: NOTIFY_EMAIL_FAIL,
            $class: 'StringParameterValue'
        ]
    ]

)

currentBuild.displayName = "lab2 @ ${demo.displayName}"
currentBuild.result = demo.result
