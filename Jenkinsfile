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
                            kubectl apply -f namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g"  values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }

        stage('Check Status'){
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1'){
                        def deploymentStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue -n $PROJECT --timeout=300s || echo FAILED" ).trim()

                        if(deploymentStatus.contains("successfully rolled out")){
                            echo "Deployment is Success"
                        }
                        else{
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                                sleep 20
                            """
                            def rollbackStatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue -n $PROJECT --timeout=300s || echo FAILED" ).trim()

                            if(rollbackStatus.contains("successfully rolled out")){
                                error "Deployment is Failed. Rollback is success"
                            }
                            else{
                                error "Deployment is Failed. ROllback also Failed. Application not running"
                            }
                        }
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