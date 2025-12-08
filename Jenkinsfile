pipeline {
  agent any
  parameters {
    credentials credentialType: 'com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl', defaultValue: 'stack_prog_aut', name: 'stack_prog_aut', required: false
    credentials credentialType: 'com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl', defaultValue: 'stack_prog', description: 'stack_prog user access keys', name: 'stack_prog', required: false
  }

  environment {
    PATH = "${PATH}:${getTerraformPath()}"
  }

  stages {
    stage('Initial Deployment Approval') {
      steps {
        script {
          def userInput = input(id: 'initial_confirm', message: 'Start Pipeline?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Start Pipeline', name: 'confirm'] ])
        }
      }
    }

    stage('terraform init') {
      steps {
        sh 'terraform init'
      }
    }

    stage('terraform plan'){
      steps {
        // Bind credentials specific to this stage's execution
        withCredentials([
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AUTO_USER_CREDS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'],
          [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.MGMT_USER_CREDS_ID, accessKeyVariable: 'AWS_ACCESS_KEY_ID_SHARED', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY_SHARED']
        ]) {
          sh 'terraform plan -out=tfplan -input=false'
        }
      }
    }
    

    //   steps {
    //     withAWS(credentials: 'stack_prog_aut', region: 'us-east-1') {
    //       sh 'terraform plan -out=tfplan -input=false'
    //     }
    //   }
    // }
    
    stage('Final Deployment Approval') { 
      steps { 
        script { 
          def userInput = input(id: 'final_confirm', message: 'Apply Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ]) 
        } 
      } 
    }

    stage('Terraform Apply'){ 
      steps {
        sh "terraform apply -input=false tfplan" 
      } 
    }
  }
}

def getTerraformPath() {
  def tfHome = tool name: 'terraform-14', type: 'terraform'
  return tfHome
}