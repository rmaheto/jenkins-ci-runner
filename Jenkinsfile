pipeline {
    agent any

    parameters {
        string(name: 'SERVICE_REPO', defaultValue: 'ssh://git@git.example.com/project/service.git', description: 'Git repo of the microservice')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        string(name: 'GIT_CREDENTIALS', defaultValue: 'service-ssh-key', description: 'Credentials ID for Git checkout')
        string(name: 'BUILD_AGENT', defaultValue: 'Linux-RHEL7-Shared-BuildAgent', description: 'Jenkins node label')
        choice(name: 'BUILD_TYPE', choices: ['build', 'build_publish', 'build_publish_deploy'], description: 'Select which pipeline stages to run')
    }

    stages {
        stage('Checkout Service Repo') {
            steps {
                dir('app') {
                    git branch: params.BRANCH_NAME,
                        credentialsId: params.GIT_CREDENTIALS,
                        url: params.SERVICE_REPO
                }
            }
        }

        stage('Run Pipeline Logic') {
            steps {
                script {
                    def pipeline = load 'app/build-pipeline.groovy'
                    pipeline.runPipeline([
                        repo: params.SERVICE_REPO,
                        branch: params.BRANCH_NAME,
                        gitCredentialsId: params.GIT_CREDENTIALS,
                        agent: params.BUILD_AGENT,
                        buildType: params.BUILD_TYPE
                    ])
                }
            }
        }
    }
}
