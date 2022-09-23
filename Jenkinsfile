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
             git url: 'https://github.com/microservices-demo/front-end.git'
                   }
        }
        stage('Sonarqube analysis') {
          steps {
            withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar') {
            sh ''' /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube_scanner/bin/sonar-scanner \

          # Définit le nom et clé du projet générée de SonarQube.
            -Dsonar.projectKey=sock-shop  \
            -Dsonar.projectName=sock-shop  \
          # Définit l’emplacement des sources que SonarQube va analyser.
            -Dsonar.sources=/var/lib/jenkins/workspace/Pipeline-microservices \
            
            -Dsonar.java.binaries=./src/main   '''
            }
               timeout(time: 10, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
       }
          }
        }
           // Building Docker images
        stage('Building docker images') {
           steps{
             script {          
              manifestsImg=docker.build("manifests-image","/home/azureuser/microservices-demo/deploy/kubernetes")
            }
           }
        }
        //Uploading Docker images into Nexus Registry
        stage('Uploading to Nexus') {
         steps{  
             script {
                docker.withRegistry(registry,registryCredentials){
                 manifestsImg.push('latest')

               }
              }
           }
        }
        stage('deploy to Openshiftt'){
              steps{ 
               script {
                sh 'oc login --insecure-skip-tls-verify  https://102.37.157.97:8443 --token=lhe8LJqSeDb-E4etZ55I80fmEPyAGQOi3TsQ-5EG7SY'
                sh 'oc project sock-shop'
                sh 'kubectl apply -f /home/azureuser/microservices-demo/deploy/kubernetes/complete-demo.yaml'
                }
           }
        }
    }
}
