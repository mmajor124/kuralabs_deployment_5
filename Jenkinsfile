pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }
        stages {
            stage ('Build'){
                steps {
                    sh '''#!/bin/bash
                    python3 -m venv test3
                    source  test3/bin/activate
                    pip install pip --upgrade
                    pip install -r requirements.txt
                    export FLASK_APP=application
                    flask run &
                    '''
                }
            }
            
            stage ('Test'){
                steps {
                    sh '''#!/bin/bash
                    source test3/bin/activate
                    py.test --verbose --junit-xml  test-reports/results.xml
                    '''
                }
                post {
                    always {
                        junit 'test-reports/results.xml'
                    }
                }
            }

            stage ('Build_And_PushDocker'){
                agent{label 'docker_agent'}
                steps {
                    sh '''#!/bin/bash
                    sudo docker build -t mmajor124/urlapp:latest .
                    echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    sudo docker push mmajor124/urlapp:latest
                    docker logout 
                    '''
                }
            }
        
            stage ('Terraform_Init'){
                agent{label 'terraform_agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform init' 
                                        }
                    }
                }
            }

            stage ('Terraform_Plan'){
                agent{label 'terraform_agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform plan -out plan.tfplan -var="aws_access_key=${aws_access_key}" -var="aws_secret_key=${aws_secret_key}"' 
                                        }
                    }
                }
            }
            
            stage ('Terraform_Apply'){
                agent{label 'terraform_agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform apply plan.tfplan' 
                                        }
                    }
                }
            }

    }
}
