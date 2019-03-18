properties(
    [
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '',
                numToKeepStr: '360')
        ),
        disableConcurrentBuilds()
                [
            $class: 'ParametersDefinitionProperty',
            parameterDefinitions: [
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
    parameters: [
        [
            name: 'OWNER_NUMBER',
            value: OWNER_NUMBER,
            $class: 'StringParameterValue',
            defaultValue: AWS_OWNER_NUMBER
        ],
        [
            name: "NOTIFY_EMAIL_PASS",
            value: NOTIFY_EMAIL_PASS,
            $class: 'StringParameterValue',
        ],
        [
            name: "NOTIFY_EMAIL_FAIL",
            value: NOTIFY_EMAIL_FAIL,
            $class: 'StringParameterValue',
        ],
    ]

)
    
currentBuild.displayName = "lab1 @ ${demo.displayName}"
currentBuild.result = demo.result
