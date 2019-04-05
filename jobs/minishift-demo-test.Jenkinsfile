// setup up and run the simple demo on four environments
//
// centos/minishift/KVM
// centos/minishift/VirtualBox


// Stage 1: Install Virtualization

// Stage 2: Install VM

// Stage 3: Install Kubevirt

// Stage 4: Execute Demos


properties(
    [
        buildDiscarder(
            logRotator(
                artifactDaysToKeepStr: '',
                artifactNumToKeepStr: '',
                daysToKeepStr: '5',
                numToKeepStr: '10'
            )
        ),
        disableConcurrentBuilds(),
        [
            $class: 'ParametersDefinitionProperty',
            parameterDefinitions: [
                [
                    name: 'TARGET_NODE',
                    description: 'Jenkins agent node',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: 'kubevirt'
                ],
                [
                    name: 'GITHUB_OWNER',
                    description: 'Github ownerfor repos',
                    $class: 'hudson.model.ChoiceParameterDefinition',
                    choices: [
                        "markllama"
                    ].join("\n"),
                    defaultValue: 'markllama'
                ],
                [
                    name: 'SSH_KEY_ID',
                    description: 'SSH credential id to use',
                    $class: 'hudson.model.ChoiceParameterDefinition',
                    choices: [
                        "markllama"
                    ].join("\n"),
                    defaultValue: 'markllama'
                ],
                [
                    name: 'MINISHIFT_VERSION',
                    description: 'What version of minishift to use (no v prefix!)',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: '1.33.0'
                ],

                [
                    name: 'MINISHIFT_GITHUB_API_TOKEN',
                    description: 'A Github API access token',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: ''
                ],
                [
                    name: 'VIRT_DRIVER',
                    description: 'Which virtualization driver to use',
                    $class: 'hudson.model.ChoiceParameterDefinition',
                    choices: [
                        "kvm",
                        "virtualbox"
                    ].join("\n"),
                    defaultValue: 'kvm'
                ],                
                [
                    name: 'OPENSHIFT_VERSION',
                    description: 'What version of open to use (no v prefix!)',
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: '3.11.0'
                ],
                [
                    name: 'START_MINISHIFT',
                    description: 'start minishift after installing',
                    $class: 'hudson.model.BooleanParameterDefinition',
                    defaultValue: true
                ],
                [
                    name: "KUBEVIRT_VERSION",
                    description: "Version of kubevirt to install (or 'none')",
                    $class: 'hudson.model.StringParameterDefinition',
                    defaultValue: "none"
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
                    description: 'leave the minishift service in place',
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
        ],
        disableConcurrentBuilds()
    ]
)

start_minishift_enabled = START_MINISHIFT.toBoolean()
persist = PERSIST.toBoolean()
debug = DEBUG.toBoolean()

def verify_github_api_access() {
    echo "verifying Github API access"
}

def get_running_vms() {
    // get the list of running machines
        // get the list of running machines
    switch (VIRT_DRIVER) {
        case 'kvm':
            machines = sh(
                returnStdout: true,
                script: "virsh --connect qemu:///system --readonly --quiet list --name"
            ).tokenize()
            break;
            
        case 'virtualbox':
            machines = sh(
                returnStdout: true,
                script: "vboxmanage list vms"
            ).readLines().collect { it.split().head() }
            break;
    }    

    echo "Machines = ${machines}"
    return machines
}

// def install_kubectl() {
//     KUBE_VERSION=sh(
//         returnStdout: true,
//         script:"curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt"
//     ).trim()
//     sh "curl -L https://storage.googleapis.com/kubernetes-release/release/${KUBE_VERSION}/bin/linux/amd64/kubectl -o ${WORKSPACE}/bin/kubectl"
//     sh "chmod a+x ${WORKSPACE}/bin/kubectl"
// }

// def get_openshift_client_tools() {
//     TARBALL_FILENAME=sh(
//         returnStdout: true,
//         script: "curl --silent --location https://github.com/openshift/origin/releases/download/v3.11.0/CHECKSUM | grep -E 'openshift-origin-client-tools-.*-linux-64bit.tar.gz' | awk '{print \$2}'"
//     ).trim()

//     sh("curl --silent --location --remote-name https://github.com/openshift/origin/releases/download/v3.11.0/${TARBALL_FILENAME}")
//     sh("tar -xzf ${TARBALL_FILENAME}")
//     TARBALL_DIRNAME=TARBALL_FILENAME.minus(".tar.gz")
//     sh("cp ${TARBALL_DIRNAME}/{kubectl,oc} ${WORKSPACE}/bin")
//     sh("chmod a+x ${WORKSPACE}/bin/{kubectl,oc}")
    
// }

def get_kvm_driver() {
    sh("curl --silent --location https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7 -o ${WORKSPACE}/bin/docker-machine-driver-kvm")
    sh("chmod +x ${WORKSPACE}/bin/docker-machine-driver-kvm")
}

def get_minishift() {
    echo "get_minishift"
    sh("curl --silent --location --remote-name https://github.com/minishift/minishift/releases/download/v${MINISHIFT_VERSION}/minishift-${MINISHIFT_VERSION}-linux-amd64.tgz")
    sh("tar -xzf minishift-${MINISHIFT_VERSION}-linux-amd64.tgz minishift-${MINISHIFT_VERSION}-linux-amd64/minishift")
    sh("mv minishift-${MINISHIFT_VERSION}-linux-amd64/minishift ${WORKSPACE}/bin")
    sh ("chmod a+x ${WORKSPACE}/bin/minishift")

}

def start_minishift() {
    echo "start_minishift"
    start_log = sh(
        returnStdout: true,
        script: "${WORKSPACE}/bin/minishift start --vm-driver ${VIRT_DRIVER}"
    )

    if (start_log =~ /OpenShift server started./) {
        // check for "OpenShift server started." in stdout
        echo "Yes! it worked!"
    } else {
        echo "--- ERROR reporting startup log ---"
        echo start_log
        echo "-----------------------------"
        error "error starting minishift"
    }

    echo "--- reporting startup log ---"
    echo start_log
    echo "-----------------------------"
}

def copy_cli_client() {
    // find the oc binary in the .minishift directory
    oc_path = sh(
        returnStdout: true,
        script: "find ${MINISHIFT_HOME} -type f -name oc"
    ).trim()
    echo "Copying ${oc_path} to ${MINISHIFT_HOME}/bin"
    sh("cp ${oc_path} ${WORKSPACE}/bin")
    sh("ln -s ${WORKSPACE}/bin/oc ${WORKSPACE}/bin/kubectl")
    sh("chmod a+x ${WORKSPACE}/bin/*")
}

def login_as_admin() {
    sh "oc login -u system:admin"
}

//
// Minishift Pods
//   NOTE: Groovy map literal order is preserved
system_pod_count = [
    "openshift-apiserver-": 1,
    "kube-dns-": 1,
    "kube-proxy-": 1,
    "openshift-service-cert-signer-operator-": 1,
    "service-serving-cert-signer-": 1,
    "apiservice-cabundle-injector-": 1,
    "kube-controller-manager-localhost": 1,
    "master-etcd-localhost": 1,
    "kube-scheduler-localhost": 1,
    "master-api-localhost": 1,
    "openshift-controller-manager-": 1,
    "persistent-volume-setup-": 1,
    "openshift-web-console-operator-": 1,
    "router-1-": 1,
    "docker-registry-1-": 1,
    "webconsole-": 1
]

def wait_for_system_pods() {

    pod_data = sh(
        returnStdout: true,
        script: "${WORKSPACE}/bin/kubectl get pods --all-namespaces -o json"
    )

    pod_object = readJSON text: pod_data

    echo "There are ${pod_object.size()} pods"
}

def enable_weave_cni() {
    kubectl_version = sh(
        returnStdout: true,
        script: "kubectl version | base64 | tr -d '\n'"
    )
    sh "${WORKSPACE}/bin/kubectl apply -f \"https://cloud.weave.works/k8s/net?k8s-version=${kubectl_version}\""

}

def install_kubevirt() {

    sh "curl --silent -L -o ${WORKSPACE}/bin/virtctl https://github.com/kubevirt/kubevirt/releases/download/v${KUBEVIRT_VERSION}/virtctl-v${KUBEVIRT_VERSION}-linux-amd64"
    sh "chmod a+x ${WORKSPACE}/bin/virtctl"

    // install the kubevirt operator
    sh "oc create -f https://github.com/kubevirt/kubevirt/releases/download/v${KUBEVIRT_VERSION}/kubevirt-operator.yaml"
    
    // enable virt emulation
    sh "oc create configmap -n kubevirt kubevirt-config --from-literal debug.useEmulation=true"

    // install the kubevirt custom resource
    sh "oc create -f https://github.com/kubevirt/kubevirt/releases/download/v${KUBEVIRT_VERSION}/kubevirt-cr.yaml"

    // wait for pods to initialize
}

def clean_minishift() {
    echo "cleaning minishift"
    sh "${WORKSPACE}/bin/minishift delete -f"
}

def check_libvirt_kvm() {
    echo "check_libvirt_kvm"
    sh("sudo systemctl status libvirtd")
}

node(TARGET_NODE) {

    checkout scm
    
    currentBuild.displayName = "${currentBuild.number} - minishift-${MINISHIFT_VERSION} / ${VIRT_DRIVER}"
    //sh("echo I ran")
    //echo "I ran"

    // This might not be needed here
    // checkout scm

    //stage("verify virtualization") {
    //    check_virt_kvm()
    //}

    if (get_running_vms().contains('minishift')) {
        error("minishift VM already exists")
    }

    withEnv(
        [
            "PATH=${WORKSPACE}/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "HOME=${WORKSPACE}",
            "MINISHIFT_HOME=${WORKSPACE}",
            "KUBECONFIG=${WORKSPACE}/.kube/config",
            "MINISHIFT_GITHUB_API_TOKEN=${MINISHIFT_GITHUB_API_TOKEN}"
        ]
    ) {
        try {

            sh "pwd"
            sh "env"
            
            stage("install minishift") {

                // where to put binaries and add path
                sh "mkdir -p bin"
                
                get_minishift()

                switch(VIRT_DRIVER) {
                    case 'kvm':
                        get_kvm_driver()
                        break;
                    
                    case 'virtualbox':
                        echo "no external driver for virtualbox"
                        break

                    default:
                        echo "ERROR - invalid virtualzation driver: ${VIRT_DRIVER}"
                        break;       
                }

            }

            // stage("install kubectl") {
            //     install_kubectl()
            // }

            stage("start minishift") {
                if (start_minishift_enabled) {
                    echo "Starting minishift"
                    start_minishift()
                    copy_cli_client()
                    login_as_admin()
                    wait_for_system_pods(4, "kube-system")
                    enable_weave_cni()
                } else {
                    echo "Minishift startup disabled"
                }
            }

            stage("install kubevirt") {
                if (start_minishift_enabled && KUBEVIRT_VERSION != 'none') {
                    echo "installing kubevirt: ${KUBEVIRT_VERSION}"
                    install_kubevirt()
                    wait_for_pods(6, 'kubevirt')
                    
                } else {
                    echo "Kubevirt installation disabled"
                }
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
            
            stage("run demo") {
                echo "running ${DEMO_NAME} from ${DEMO_GIT_REPO}:${DEMO_GIT_BRANCH}"

                sh "which kubectl"

                def filename = "demo-test-result-${demo_name}.txt"


                return_code = sh (
                    returnStatus: true,
                    script: "scripts/run_demo.py -d -t demos/${DEMO_ROOT}/${DEMO_NAME} -o ${filename} 2>&1"
                )

                if (return_code != 0) {
                    currentBuild.result = "FAILURE"
                }

                def result = readFile file: filename
                echo "result = --- \n${result}\n---"
            }
            
        } finally {
            stage("teardown minishift") {

                archiveArtifacts artifacts: "demo-test-result-*.txt"

                if (!persist) {
                    echo "Cleaning up minishift on agent"
                    try {
                        clean_minishift()
                    } catch (err) {
                        echo "error cleaning minishift"
                    }
                    cleanWs()
                    deleteDir()
                } else {
                    echo "PERSIST = true - cleanup disabled"
                }
            }
        }
    }
}
