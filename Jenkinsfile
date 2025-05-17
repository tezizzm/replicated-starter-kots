pipeline {
  agent { label 'docker-agent' }

  parameters {
    string(
      name: 'RELEASE_BRANCH',
      defaultValue: "${env.BRANCH_NAME}",
      description: 'Git branch to release from'
    )
    string(
      name: 'RELEASE_TAG',
      defaultValue: "${env.TAG_NAME}",
      description: 'Git tag for this release (only triggers release stages when non-empty)'
    )
  }

  environment {
    REPLICATED_API_TOKEN = credentials('replicated-api-token')
    REPLICATED_APP       = credentials('replicated-app')
    REPLICATED_CLI_IMAGE = 'replicated/vendor-cli:latest'
  }

  stages {
    stage('Validate Inputs') {
      steps {
        echo "Branch: ${params.RELEASE_BRANCH}"
        echo "Tag:    ${params.RELEASE_TAG}"
        echo "App ID: ${env.REPLICATED_APP}"
      }
    }

    stage('Create Release') {
      when {
        expression { params.RELEASE_TAG?.trim() }
      }
      steps {
        container('docker') {
          dir("${env.WORKSPACE}") {
            sh """
              docker run --rm \
                -e REPLICATED_API_TOKEN=${env.REPLICATED_API_TOKEN} \
                -e REPLICATED_APP=${env.REPLICATED_APP} \
                -e GITHUB_BRANCH_NAME=${params.RELEASE_BRANCH} \
                -e GITHUB_TAG_NAME=${params.RELEASE_TAG} \
                ${env.REPLICATED_CLI_IMAGE} \
                release create --auto -y
            """
          }
        }
      }
    }

    stage('Release Kubernetes Installer') {
      when {
        expression { params.RELEASE_TAG?.trim() }
      }
      steps {
        container('docker') {
          dir("${env.WORKSPACE}") {
            sh """
              docker run --rm \
                -e REPLICATED_API_TOKEN=${env.REPLICATED_API_TOKEN} \
                -e REPLICATED_APP=${env.REPLICATED_APP} \
                -e GITHUB_BRANCH_NAME=${params.RELEASE_BRANCH} \
                -e GITHUB_TAG_NAME=${params.RELEASE_TAG} \
                ${env.REPLICATED_CLI_IMAGE} \
                installer create --auto -y
            """
          }
        }
      }
    }
  }

  post {
    always {
      // Clean workspace using the Workspace Cleanup Plugin
      cleanWs(
        deleteDirs: true,                    // delete directories as well as files
        patterns: [[pattern: '**/*', type: 'INCLUDE']],
        failOnError: false                   // don’t fail the build if cleanup errors occur
      )
    }
    success {
      echo '✅ Pipeline completed successfully.'
    }
    failure {
      echo '🚨 Pipeline failed!'
    }
  }
}