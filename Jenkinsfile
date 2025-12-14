pipeline {
  agent any

  parameters {
    credentials credentialType: 'com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl', defaultValue: 'stack_prog_aut', name: 'AWS', required: false
    booleanParam(name: 'BUILD_AMI', defaultValue: false)
    booleanParam(name: 'DESTROY', defaultValue: false)
    string defaultValue: 'DEJI', name: 'RUNNER'
  }

  environment {
    PATH = "${PATH}:${getTerraformPath()}"
  }

  stages {
    stage('terraform init') {
      steps {
        slackSend (color: '#d0ff00ff', message: "${params.RUNNER} STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        sh 'terraform init'
      }
    }

    stage('terraform plan and apply'){
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: params.AWS, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
        ]) {
          slackSend (color: '#15ff00ff', message: "GENERATING PLAN: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          sh 'terraform plan -out=tfplan -input=false'
          sh "terraform apply -input=false tfplan" 
          slackSend (color: '#0400ffff', message: "FINISHED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        } 
      }
    }

    stage('terraform destroy'){
      when {
        expression { params.DESTROY }
      }
      steps {
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: params.AWS, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
        ]) {
          sh "terraform destroy -auto-approve"
          slackSend (color: '#ff000dff', message: "DESTROYED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Initiated by ${params.RUNNER}")
        }
      }
    }
  }
}

def getTerraformPath() {
  def tfHome = tool name: 'terraform-14', type: 'terraform'
  return tfHome
}