properties(
    [
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '5',
                artifactNumToKeepStr: '10',
                daysToKeepStr: '10',
                numToKeepStr: '5')
        ),
        disableConcurrentBuilds(),
        [
            $class: 'ParametersDefinitionProperty',
            parameterDefinitions: [
                [
                    name: 'TARGET_NODE',
                    description: 'Jenkins agent node',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'gcloud'
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
                    defaultValue: "https://github.com/markllama/kubevirt.github.io.git"
                ],
                [
                    name: "DEMO_GIT_BRANCH",
                    description: "The branch that contains the of the demo to run",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "labs"
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
node(TARGET_NODE) {
    stage("run lab1 on GCP") {
        demo = build(
            job: "kubevirt/gcp-demo-test",
            propagate: false,
            parameters: [
                [
                    name: "TARGET_NODE",
                    value: TARGET_NODE,
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

        copyArtifacts(
            projectName: 'kubevirt/gcp-demo-test',
            selector: specific("${demo.number}")
        )

        archiveArtifacts artifacts: "demo-test-result-*.txt"

        currentBuild.displayName = "lab1@gcp ${demo.displayName}"
        currentBuild.result = demo.result

    }
}
