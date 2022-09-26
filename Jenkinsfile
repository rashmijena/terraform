pipeline {
    agent any

    environment {
        //AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        //AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_ACCESS_KEY_ID     = 'AKIASWW7QAAUNK6A6URC'
        AWS_SECRET_ACCESS_KEY = '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'
        AWS_DEFAULT_REGION    = "eu-west-1"
        BACKEND_BUCKET        = "jenkins-terraform-pipeline-state"
    }

    parameters {
        string(name:'envTarget', defaultValue:'DEV',description:'currently defaulting to DEV')
        string(name:'subEnvTarget', defaultValue:'HOM',description:'currently defaulting to HOM')
        choice(
            name: 'Action',
            choices: ['Build', 'Destroy'],
            description: 'The action to take'
        )
        choice(
            name: 'Colour',
            choices: ['Blue', 'Green'],
            description: 'The environment to use'
        )
    }

    stages {
        stage('Init') {
            steps {
                terraformInit()
            }
        }
        stage('Plan') {
            steps {
                terraformPlan()
            }
        }
        stage('Approval') {
            steps {
                input(message: 'Apply Terraform ?')
            }
        }
        stage('Apply') {
            steps {
                terraformApply()
            }
        }
        stage('Validate') {
            steps {
                inspecValidation()
            }
        }
    }
    post {
        always {
            echo 'Deleting Directory!'
            deleteDir()
        }
    }
}

def terraformInit() {
    sh("""
        cd DEV/HOM;
        terraform init -backend-config="bucket=${env.BACKEND_BUCKET}" -backend-config="key=demo.tfstate"
        terraform workspace select ${params.Colour.toLowerCase()} || terraform workspace new ${params.Colour.toLowerCase()}
    """)
}

def terraformPlan() {
    // Setting Terraform Destroy flag
    if(params.Action == 'Destroy') {
        env.DESTROY = '-destroy'
    } else {
        env.DESTROY = ""
    }

    sh("""
        cd DEV/HOM;
        terraform plan ${env.DESTROY} -var-file=${params.Colour.toLowerCase()}.tfvars -no-color -out=tfout
    """)
}

def terraformApply() {
    sh("""
        cd DEV/HOM;
        terraform apply tfout -no-color
        mkdir ../../Inspec/files/
        terraform output --json > ../../Inspec/files/output.json
    """)
}

def inspecValidation() {
    sh("""
        inspec exec Inspec/ -t aws:// --input workspace=${params.Colour}
    """)
}

def locateEnvFolder() {
    return params.envTarget + '/' + params.subEnvTarget
}