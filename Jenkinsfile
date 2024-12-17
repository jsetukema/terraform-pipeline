pipeline {

    agent{ label '!master' }

    parameters {
        string(name: 'AppName', defaultValue: 'Enter App Name', description: 'Name of application', )

        choice(choices: ['master', 'dev', 'qa', 'prod'], description: 'Select lifecycle to Deploy', name: 'Branch')

        choice(choices: ['t2.micro', 't2.small', 't2.medium', 't2.large'], description: 'Select Instance Size', name: 'InstanceSize')

        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }
     environment {

        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')

        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')

        TF_VAR_instance_type = "${params.InstanceSize}"

        TF_VAR_environment = "${params.Branch}"

        TF_VAR_application = "${params.AppName}"
    }
//
    stages {

      stage('checkout') {

        steps {

            echo "Pulling changes from the branch ${params.Branch}"

            git credentialsId: '715c9bc9-c0a7-42b7-94a9-a5a899ba1694', url: 'https://github.com/jsetukema/terraform-pipeline.git', branch: "${params.Branch}"

        }
      }
        stage('terraform plan') {

            steps {

                sh "pwd ; terraform init -input=true"

                sh "terraform plan -input=true -out tfplan"

                sh 'terraform show -no-color tfplan > tfplan.txt'
}

            }

        stage('terraform apply approval') {

           when {
               not {

                   equals expected: true, actual: params.autoApprove
               }

           }

           steps {
               script {
                    def plan = readFile 'tfplan.txt'

                    input message: "Do you want to apply the plan?",

                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('terraform apply') {
            steps {
                sh "terraform apply -input=true tfplan"
            }
        }
        stage('terraform destroy approval') {

            steps {
                input 'Run terraform destroy?'
            }
        }
        stage('terraform destroy') {

            steps {
                sh 'terraform destroy -auto-approve'
            }
        }
    }
  }