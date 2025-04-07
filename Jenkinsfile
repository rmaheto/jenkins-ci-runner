pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        string(name: 'BUILD_AGENT', defaultValue: 'Linux-RHEL7-Shared-BuildAgent', description: 'Agent label')
        choice(name: 'BUILD_TYPE', choices: ['build', 'build_publish', 'build_publish_deploy'], description: 'What to run')
    }

    environment {
        SERVICE_REPO     = "${SERVICE_REPO ?: 'ssh://git@github.com/your-org/placeholder.git'}"
        SERVICE_NAME     = "${SERVICE_NAME ?: 'unknown-service'}"
        GIT_CREDENTIALS  = "${GIT_CREDENTIALS ?: 'default-ssh'}"
    }

    stages {
        stage("Checkout ${env.SERVICE_NAME}") {
            steps {
                dir("app") {
                    git branch: params.BRANCH_NAME,
                        credentialsId: env.GIT_CREDENTIALS,
                        url: env.SERVICE_REPO
                }
            }
        }

        stage('Run Service Pipeline') {
            steps {
                script {
                    def inputProps = readJSON file: "app/ci/input.json"

                    def pipelineScript = load "app/ci/build-pipeline.groovy"

                    pipelineScript.runPipeline([
                        branch: params.BRANCH_NAME,
                        gitCredentialsId: env.GIT_CREDENTIALS,
                        agent: params.BUILD_AGENT,
                        buildType: params.BUILD_TYPE,
                        repo: env.SERVICE_REPO,
                        props: inputProps
                    ])
                }
            }
        }
    }
}
