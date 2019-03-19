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
                    name: "KUBECTL_PATH",
                    description: "The path of a directory containing the kubectl binary",
                    $class: 'hudson.model.StringParameterDefinition',
                ],
                [
                    name: "KUBECONFIG",
                    description: "The location of the kubeconfig file",
                    $class: 'hudson.model.StringParameterDefinition',
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

node(TARGET_NODE) {

    checkout scm

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

    withEnv(
        [
            "PATH=${KUBECTL_PATH}:${PATH}",
            "KUBECONFIG=${KUBECONFIG}"
        ]
    ) {

        stage("execute test") {
            echo "execute test"

            def filename = "demo-test-result-${demo_name}.txt"
            
            return_code = sh(
                returnStatus: true,
                script: "scripts/run_demo.py -d -t demos/${DEMO_ROOT}/${DEMO_NAME} -o ${filename}"
            )
       
//            result = sh (
//                returnStdout: true,
//                script: "scripts/run_demo.py -d -t demos/${DEMO_ROOT}/${DEMO_NAME}"
//            )

            def result = readFile :file filename
            echo "result = --- \n${result}\n---"
            
            // writeFile(
            //    file: "demo-test-result-${demo_name}.txt",
            //    text: "${result}"
            // )
        }
    }

    archiveArtifacts artifacts: "demo-test-result-*.txt"



    if (!persist) {
        cleanWs()
        deleteDir()
    } 
    
}

