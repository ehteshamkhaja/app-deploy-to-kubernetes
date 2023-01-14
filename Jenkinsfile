pipeline {

    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    stages{

        stage('Sonar Quality Status'){
         agent {
            docker {
                image 'maven'
            }
         }
         steps{
            script{
                withSonarQubeEnv(credentialsId: 'sonar-token') {
    
                sh 'mvn clean package sonar:sonar'
              }
            }
           }
         }
         stage('Quality Gate Status'){
            steps{

                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
         }
        stage('Docker Build and Push Image to Nexus'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]) {
    
                    sh '''
                    docker build -t 34.221.13.52:8083/springapp:${VERSION} .
                    docker login -u admin -p $nexus_creds 34.221.13.52:8083
                    docker push 34.221.13.52:8083/springapp:${VERSION}
                    docker rmi 34.221.13.52:8083/springapp:${VERSION}
                    ''' 
                }  
                }
            }
        }
        stage('Pushing the helm charts to nexus repo'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_passwd', variable: 'nexus_creds')]){
                    dir('kubernetes/') {
                    sh '''
                    helmversion=$(helm show chart myapp | grep version | cut -d':' -f2 | tr -d ' ' )
                    tar -czvf myapp-${helmversion}.tgz myapp/
                    curl -u admin:$nexus_creds http://34.221.13.52:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                     
                    '''
                    }
                }
            }
        }
        }
        
    }    
}
