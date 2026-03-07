pipeline{
    // agent any
    agent {
        node {
            label 'AGENT-1'
        }
    }
    // Pre-Build
    environment{
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "235270182665"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image Version of the application')
        //string(name: 'imageURL', description: 'Image URL of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Choose the Environment')
    } 
    
   // Build
    stages{
        stage('Deploy'){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                        """
                    }
                }
            }
        }
    }

   // Post-Build
    post { 
        always { 
            echo 'I will always say Hello pipeline!'
            deleteDir()
        }
        success { 
            echo 'Hello pipeline is success!'
        }
        failure { 
            echo 'Hello pipeline is failure!'
        }
    }
}