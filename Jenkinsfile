pipeline {
  agent any

  parameters {
    choice choices: ['apply', 'destroy'], name: 'DEPLOY'
    // booleanParam(name: 'DESTROY', defaultValue: false)
    string defaultValue: 'DEJI', name: 'RUNNER'
  }

  environment {
    PATH = "${PATH}:${getTerraformPath()}"
  }

  stages {
    stage('Terraform init') {
      steps {
        slackSend (color: '#ff9900ff', message: "${params.RUNNER} STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        sh 'terraform init'
      }
    }

    stage('Terraform Validate') {
      steps {
        slackSend (color: '#d9ff00ff', message: "${params.RUNNER} VALIDATED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        sh 'terraform validate'
      }
    }

    stage('Terraform Plan'){
      steps {
        slackSend (color: '#15ff00ff', message: "GENERATING PLAN: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        sh 'terraform plan -out=tfplan -input=false'
      }
    }

    stage('Terraform Apply'){
      steps {
        script {
          if (params.DEPLOY == 'apply') {
            sh "terraform apply -input=false tfplan" 
            slackSend (color: '#0400ffff', message: "FINISHED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }
          else if (params.DEPLOY == 'destroy') {
            sh "terraform destroy -auto-approve"
            slackSend (color: '#ff000dff', message: "DESTROYED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Initiated by ${params.RUNNER}")
          }
        }
      }
    }
    // stage('Terraform Destroy'){
    //   when {
    //     expression { params.DESTROY }
    //   }
    //   steps {
    //     sh "terraform destroy -auto-approve"
    //     slackSend (color: '#ff000dff', message: "DESTROYED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Initiated by ${params.RUNNER}")
    //   }
    // }
  }
}

def getTerraformPath() {
  def tfHome = tool name: 'terraform-14', type: 'terraform'
  return tfHome
}