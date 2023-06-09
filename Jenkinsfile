pipeline{
    
    agent any 

    environment{
        AWS_ACCOUNT_ID="p5u5p5h0"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="buchananecr"
        REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/eliyazsyed22/buchananCIandCD.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Maven Build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }
        stage('Static code Analysis'){
            
            steps{
                script{
                   withSonarQubeEnv(credentialsId: 'Sonarqube-token') {
                        sh "mvn clean package sonar:sonar"
                    }
                }
            }
        }
        stage('Quality Gate'){
            
            steps{
                script{
                   waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube-token'
                }
            }
        }
        /*stage('Upload Artifact to Nexus Repo'){
            
            steps{
                script{
                   nexusArtifactUploader artifacts: 
                   [
                        [   artifactId: 'springboot', 
                            classifier: '', file: 'target/Uber.jar', 
                            type: 'jar'
                        ]
                    ], 
                    credentialsId: 'BT-nexus', 
                    groupId: 'com.example', 
                    nexusUrl: '65.2.39.123:8081',
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'buchananrepo-release', 
                    version: '1.0.0'
                }
            }
        }*/
         stage('Elastic Container Registry Login'){
            
            steps{
                script{
                   sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/p5u5p5h0'
                }
            }
        }
        stage('Docker Image Build'){
            
            steps{
                script{
                   //sh 'docker image build -t buchanan:v1.$BUILD_ID .'
                   sh 'docker build -t buchananecr:buchananlatest .'
                   sh 'docker tag buchananecr:buchananlatest public.ecr.aws/p5u5p5h0/buchananecr:buchananlatest'
                }
            }
        }
        stage('Docker Image Push to ECR'){

            steps{
                script{
                   sh 'docker push public.ecr.aws/p5u5p5h0/buchananecr:buchananlatest'
                }
            }
        }  
          stage('K8s Deployment to EKS cluster'){

            steps{
                    withKubeConfig(caCertificate: '', 
                                   clusterName: '', 
                                   contextName: '', 
                                   credentialsId: 'btconfigeks', 
                                   namespace: '', restrictKubeConfigAccess: false, 
                                   serverUrl: '') 
                                /*{
                                    sh 'kubectl apply -f eks-deploy-k8s.yaml -n buchanan'
                                    sh 'kubectl apply -f eks-test-k8s.yaml -n buchanan'
                                }*/
            }
        }            
    }       

}