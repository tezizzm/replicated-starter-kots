pipeline {
  agent { label 'replicated-agent' }

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
    GITHUB_BRANCH_NAME   = "${params.RELEASE_BRANCH}"
    GITHUB_TAG_NAME      = "${params.RELEASE_TAG}"
  }

  stages {
    stage('Validate Inputs') {
      steps {
        echo "Branch: $GITHUB_BRANCH_NAME"
        echo "Tag:    $GITHUB_TAG_NAME"
        echo "App ID: $REPLICATED_APP"
      }
    }

    stage('Create Release') {
      when {
        expression { env.GITHUB_TAG_NAME?.trim() }
      }
      steps {
        container('replicated-cli') {
          dir("${env.WORKSPACE}") {
            sh 'release create --auto -y'
          }
        }
      }
    }

    stage('Release Kubernetes Installer') {
      when {
        expression { env.GITHUB_TAG_NAME?.trim() }
      }
      steps {
        container('replicated-cli') {
          dir("${env.WORKSPACE}") {
            sh 'installer create --auto -y'
          }
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
    success {
      echo '✅ Pipeline completed successfully.'
    }
    failure {
      echo '🚨 Pipeline failed!'
    }
  }
}