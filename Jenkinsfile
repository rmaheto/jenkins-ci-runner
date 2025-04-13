pipeline {
	agent any

    parameters {
		string(name: 'BRANCH_NAME', defaultValue: 'master', description: 'Branch to build')
        string(name: 'BUILD_AGENT', defaultValue: 'Linux-RHEL7-Shared-BuildAgent', description: 'Jenkins node label')
        choice(name: 'BUILD_TYPE', choices: ['build', 'build_publish', 'build_publish_deploy'], description: 'Build type')
    }

    environment {
		CHECKOUT_DIR = 'authentication-service'
    }

    stages {
		stage('Initialize and Launch Service Pipeline') {
			steps {
				script {
					// --- Derive Config ---
                    def jobName = env.JOB_NAME.split('/').last()
                    def serviceName = jobName.replaceFirst(/^CI-/, '')
                    env.SERVICE_NAME = serviceName
                    env.SERVICE_REPO = "git@github.com:rmaheto/${serviceName}.git"
                    env.GIT_CREDENTIALS = "github-ssh-key"

                    echo "🔧 Derived SERVICE_REPO: ${env.SERVICE_REPO}"
                    echo "🔐 Using Git credentials ID: ${env.GIT_CREDENTIALS}"

                    // --- Checkout ---
                    echo "🔄 Cloning repo into ${env.CHECKOUT_DIR}..."
                    dir(env.CHECKOUT_DIR) {
						checkout([
                            $class: 'GitSCM',
                            branches: [[name: "*/${params.BRANCH_NAME}"]],
                            userRemoteConfigs: [[
                                url: env.SERVICE_REPO,
                                credentialsId: env.GIT_CREDENTIALS
                            ]],
                            extensions: [[$class: 'CleanBeforeCheckout']]
                        ])
                    }

                    // --- Load Config ---
                    def inputProps = readJSON file: "${env.CHECKOUT_DIR}/input.json"
                    env.SOLUTION_ID = inputProps.SOLUTION_ID ?: 'unknown'
                    env.APPLICATION = inputProps.APPLICATION ?: env.SERVICE_NAME
                    env.PIPELINE_PROPS = inputProps.collectEntries { k, v -> [(k): v.toString()] }

                    echo "📦 Loaded config for ${env.APPLICATION} (SOLUTION_ID: ${env.SOLUTION_ID})"

                    // --- Run Service Pipeline ---
                    def fullCheckoutDir = "${pwd()}/${env.CHECKOUT_DIR}"
                    def pipelineScript = load "${fullCheckoutDir}/build-pipeline.groovy"

                    echo "🚀 Executing service pipeline..."
                    pipelineScript.runPipeline([
                        branch          : params.BRANCH_NAME,
                        gitCredentialsId: env.GIT_CREDENTIALS,
                        agent           : params.BUILD_AGENT,
                        buildType       : params.BUILD_TYPE,
                        repo            : env.SERVICE_REPO,
                        props           : inputProps,
                        serviceName     : env.SERVICE_NAME,
                        dir             : fullCheckoutDir
                    ])
                }
            }
        }
    }
}
