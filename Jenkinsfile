pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        string(name: 'BUILD_AGENT', defaultValue: 'Linux-RHEL7-Shared-BuildAgent', description: 'Jenkins node label')
        choice(name: 'BUILD_TYPE', choices: ['build', 'build_publish', 'build_publish_deploy'], description: 'Build type')
    }

    environment {
        APP_DIR = 'app'
    }

    stages {
        stage('Derive SERVICE_REPO from Job Name') {
            steps {
                script {
                    def jobName = env.JOB_NAME.split('/').last() // Support foldered jobs
                    def serviceName = jobName.replaceFirst(/^CI-/, '')
                    env.SERVICE_NAME = serviceName
                    env.SERVICE_REPO = "git@github.com:rmaheto/${serviceName}.git"

                    echo "Derived SERVICE_REPO: ${env.SERVICE_REPO}"
                }
            }
        }

        stage('Initial Checkout') {
            steps {
                dir(env.APP_DIR) {
                    git branch: params.BRANCH_NAME,
                        url: env.SERVICE_REPO
                }
            }
        }

        stage('Read input.json') {
            steps {
                script {
                    def inputProps = readJSON file: "${env.APP_DIR}/ci/input.json"
                    env.GIT_CREDENTIALS = inputProps.GIT_CREDENTIALS
                    env.SOLUTION_ID = inputProps.SOLUTION_ID
                    env.APPLICATION = inputProps.APPLICATION
                }
            }
        }

        stage('Re-Checkout with Credentials') {
            steps {
                dir(env.APP_DIR) {
                    deleteDir()
                    git branch: params.BRANCH_NAME,
                        url: env.SERVICE_REPO,
                        credentialsId: env.GIT_CREDENTIALS
                }
            }
        }

        stage('Run Service Pipeline') {
            steps {
                script {
                    def inputProps = readJSON file: "${env.APP_DIR}/ci/input.json"

                    def pipelineScript = load "${env.APP_DIR}/ci/build-pipeline.groovy"

                    pipelineScript.runPipeline([
                        branch: params.BRANCH_NAME,
                        gitCredentialsId: env.GIT_CREDENTIALS,
                        agent: params.BUILD_AGENT,
                        buildType: params.BUILD_TYPE,
                        repo: env.SERVICE_REPO,
                        props: inputProps,
                        serviceName: env.SERVICE_NAME
                    ])
                }
            }
        }
    }
}
