pipeline {
    agent {
        label 'AGENT-1' 
    }
   // agent any
    options{
         timeout(time: 30, unit: 'MINUTES')
         disableConcurrentBuilds()
         ansiColor('xterm')
    }
    parameters {
        choice(name: 'ACTION', choices: ['Apply', 'Destroy'], description: 'Pick something') 
    }
     environment {
        DEBUG = 'true'
        appVersion = '' // this will become global, we can use across pipeline
        region = 'us-east-1'
        account_id = '614019632196'
        project = 'expense'
        environment = 'dev'
        component = 'frontend'
    }
    stages {
        stage('Read The version') {
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App version: ${appVersion}"
                }
            }
        }    
        stage('Docker build') {
            
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-configure') {
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .
    
                    docker images

                    docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
                    """
                }
            }
        }
        stage('Deploy'){
            steps{
                 withAWS(region: 'us-east-1', credentials: 'aws-configure') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
                        helm upgrade --install ${component} -n ${project} -f values.yaml .
                    """
                }
            }
        }
        // stage('Deploy') {
        //     when {     
        //          expression {
        //            env.GIT_BRANCH = 'origin/main'  
        //         }
        //     }
        //     steps {
        //          sh "echo this is Deploy stage1"
        //     }
        // }
        
    }
    
    post {
        always{
            echo " this is always block  "
            deleteDir()
        }
        success{
            echo " this is success block  "
        }
        failure{
            echo " this is failure block  "
        }
    }
}