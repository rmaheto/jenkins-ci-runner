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
		stage('setup') {
			steps {
				script {
					def jobName = env.JOB_NAME.split('/').last()
                    def serviceName = jobName.replaceFirst(/^CI-/, '')

                    env.SERVICE_NAME = serviceName
                    env.SERVICE_REPO = "git@github.com:rmaheto/${serviceName}.git"
                    env.GIT_CREDENTIALS = "github-ssh-key"

                    echo "Derived SERVICE_REPO: ${env.SERVICE_REPO}"
                    echo "Using Git credentials ID: ${env.GIT_CREDENTIALS}"

                    echo "🔄 Cloning repo ${env.SERVICE_REPO} into ${env.CHECKOUT_DIR}..."

                    dir(env.CHECKOUT_DIR) {
						checkout([
                            $class: 'GitSCM',
                            branches: [[name: "*/${params.BRANCH_NAME}"]],
                            userRemoteConfigs: [[
                                url: env.SERVICE_REPO,
                                credentialsId: env.GIT_CREDENTIALS
                            ]],
                            doGenerateSubmoduleConfigurations: false,
                            submoduleCfg: [],
                            extensions: [[$class: 'CleanBeforeCheckout']]
                        ])
                    }


						def inputProps = readJSON file: "${env.CHECKOUT_DIR}/input.json"
                    env.SOLUTION_ID = inputProps.SOLUTION_ID ?: 'unknown'
                    env.APPLICATION = inputProps.APPLICATION ?: env.SERVICE_NAME
                    env.PIPELINE_PROPS = inputProps.collectEntries { k, v -> [(k): v.toString()] } // for reuse


                def fullCheckoutDir = "${pwd()}/${env.CHECKOUT_DIR}"
                    def inputProps = readJSON file: "${env.CHECKOUT_DIR}/input.json"
                    def pipelineScript = load "${fullCheckoutDir}/build-pipeline.groovy"

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

        //stage('Checkout with Credentials') {
		//	steps {
		//		script {
		//			echo "🔄 Cloning repo ${env.SERVICE_REPO} into ${env.CHECKOUT_DIR}..."
		//
        //            dir(env.CHECKOUT_DIR) {
		//				checkout([
        //                    $class: 'GitSCM',
        //                    branches: [[name: "*/${params.BRANCH_NAME}"]],
        //                    userRemoteConfigs: [[
        //                        url: env.SERVICE_REPO,
        //                        credentialsId: env.GIT_CREDENTIALS
        //                    ]],
        //                    doGenerateSubmoduleConfigurations: false,
        //                    submoduleCfg: [],
        //                    extensions: [[$class: 'CleanBeforeCheckout']]
        //                ])
        //            }
        //        }
        //    }
        //}

        //stage('Load Config') {
		//	steps {
		//		script {
		//			def inputProps = readJSON file: "${env.CHECKOUT_DIR}/input.json"
        //            env.SOLUTION_ID = inputProps.SOLUTION_ID ?: 'unknown'
        //            env.APPLICATION = inputProps.APPLICATION ?: env.SERVICE_NAME
        //            env.PIPELINE_PROPS = inputProps.collectEntries { k, v -> [(k): v.toString()] } // for reuse
        //        }
        //    }
        //}

        //stage('Run Service Pipeline') {
		//	steps {
		//		script {
		//			def fullCheckoutDir = "${pwd()}/${env.CHECKOUT_DIR}"
        //            def inputProps = readJSON file: "${env.CHECKOUT_DIR}/input.json"
        //            def pipelineScript = load "${fullCheckoutDir}/build-pipeline.groovy"
		//
        //            pipelineScript.runPipeline([
        //                branch          : params.BRANCH_NAME,
        //                gitCredentialsId: env.GIT_CREDENTIALS,
        //                agent           : params.BUILD_AGENT,
        //                buildType       : params.BUILD_TYPE,
        //                repo            : env.SERVICE_REPO,
        //                props           : inputProps,
        //                serviceName     : env.SERVICE_NAME,
        //                dir             : fullCheckoutDir
        //            ])
        //        }
        //    }
        //}
    }
}
