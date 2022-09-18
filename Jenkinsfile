pipeline {
    agent any
    environment{
        registryCredentials = "nexus"
        registry = "http://nexus-registry-nexus.102.37.157.97.nip.io"
        manifestsImg= ''
        SCANNER_HOME = tool 'SonarQube_scanner'
    }
    stages {
        stage('code Checkout') {
            steps {
             git url: 'https://github.com/rabeb146/microservices-demo.git'
            }
        }
        stage('Sonarqube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
            sh '''/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube_scanner/bin/sonar-scanner \
            -Dsonar.projectKey=sock-shop  \
            -Dsonar.projectName=sock-shop  \
            -Dsonar.sources=/var/lib/jenkins/workspace/Pipeline-microservices \
            -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
            }
          }
        }
           // Building Docker images
        stage('Building docker image') {
           steps{
             script {
              //sh 'make'
              manifestsImg=docker.build("manifests-image","/var/lib/jenkins/workspace/Pipeline-microservices/deploy/kubernetes/")
            }
           }
        }
        //Uploading Docker images into Nexus Registry
        stage('Uploading to Nexus') {
         steps{  
             script {
                docker.withRegistry(registry,registryCredentials){
                 //sh 'docker push nexus-registry-nexus.102.37.157.97.nip.io/manifests-image'
                 manifestsImg.push('latest')

               }
              }
           }
        }
        stage('deploy to Openshiftt'){
              steps{ 
               script {
                sh 'oc login --insecure-skip-tls-verify  https://102.37.157.97:8443 --token=kOhhAKFIiobdfN7YgFR_HaQOb8LIPZKAMy8u1trv0cY'
                // createOPNamespaceIfNotExist('sock-shop2')
                sh 'oc project sock-shop'
               // sh ' docker pull nexus-registry-nexus.102.37.157.97.nip.io/nexus-registry/manifests-image:latest'
               sh 'kubectl apply -f /home/azureuser/microservices-demo/deploy/kubernetes/complete-demo.yaml'
                }
           }
        }
    }
}
