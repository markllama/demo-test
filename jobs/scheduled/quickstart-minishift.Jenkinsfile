properties(
    [
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '30',
                artifactNumToKeepStr: '10',
                daysToKeepStr: '30',
                numToKeepStr: '10')
        ),
        disableConcurrentBuilds(),
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
                    name: 'MINISHIFT_VERSION',
                    description: 'What version of minikube to use (no v prefix!)',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: '1.33.0'
                ],
                [
                    name: 'OPENSHIFT_VERSION',
                    description: 'What version of openshift to use (no v prefix!)',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "3.11.0"
                ],
                [
                    name: 'VIRT_DRIVER',
                    description: 'Which virtualization driver to use',
                    $class: 'hudson.model.ChoiceParameterDefinition',
                    choices: [
                        "kvm2",
                        "kvm",
                        "virtualbox"
                    ].join("\n"),
                    defaultValue: 'kvm'
                ],                
                [
                    name: "KUBEVIRT_VERSION",
                    description: "Version of kubevirt to install (or 'none')",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "0.15.0"
                ],
                [
                    name: 'DEMO_NAME',
                    description: "The name of the demo to run",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'quickstart_minikube'
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
                    defaultValue: "test-quickstart"
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
    stage("run quickstart on Minishift") {
        demo = build(
            job: "kubevirt/minishift-demo-test",
            propagate: false,
            parameters: [
                [
                    name: "TARGET_NODE",
                    value: TARGET_NODE,
                    $class: 'StringParameterValue'
                ],
                [
                    name: "MINISHIFT_VERSION",
                    value: MINISHIFT_VERSION,
                    $class: 'StringParameterValue'
                ],
                [
                    name: "START_MINISHIFT",
                    value: true,
                    $class: 'BooleanParameterValue'
                ],
                [
                    name: "VIRT_DRIVER",
                    value: VIRT_DRIVER,
                    $class: 'StringParameterValue'
                ],
                [
                    name: "KUBEVIRT_VERSION",
                    value: KUBEVIRT_VERSION,
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
            projectName: 'kubevirt/minishift-demo-test',
            selector: specific("${demo.number}")
        )

        archiveArtifacts artifacts: "demo-test-result-*.txt"

        currentBuild.displayName = "quickstart@minishift ${demo.displayName}"
        currentBuild.result = demo.result

    }
}
