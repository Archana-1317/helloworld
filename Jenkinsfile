pipeline {
  agent any

  tools {
    jdk 'java8'
    maven 'Maven'
  }
  
  environment {

      sonar_url = 'http://10.128.0.3:9000'
      sonar_username = 'admin'
      sonar_password = 'admin'
      nexus_url = '10.128.0.3:8081'
      artifact_version = '4.0.0'

 }
 parameters {
      string(defaultValue: 'main', description: 'Please type any branch name to deploy', name: 'Branch')
 }  

stages {
    stage('Git checkout'){
      steps {
        git branch: '${Branch}',
        url: 'https://github.com/Archana-1317/helloworld.git'
      }
    }
    stage('Maven build'){
      steps {
        sh 'mvn clean install'
      }
    }
  stage ('Sonarqube Analysis'){
           steps {
           withSonarQubeEnv('sonarqube') {
           sh '''
           mvn -e -B sonar:sonar -Dsonar.java.source=1.8 -Dsonar.host.url="${sonar_url}" -Dsonar.login="${sonar_username}" -Dsonar.password="${sonar_password}" -Dsonar.sourceEncoding=UTF-8
           '''
           }
         }
      } 
      stage ('Publish Artifact') {
        steps {
          nexusArtifactUploader artifacts: [[artifactId: 'hello-world-war', classifier: '', file: "target/hello-world-war-1.0.0.war", type: 'war']], credentialsId: 'nexus-cred', groupId: 'com.efsavage', nexusUrl: "${nexus_url}", nexusVersion: 'nexus3', protocol: 'http', repository: 'release', version: "${artifact_version}"
        }
      }
      stage ('Build Docker Image'){
        steps {
          sh '''
          cd ${WORKSPACE}
          docker build -t gcr.io/sylvan-bonbon-357404/helloworld --file=Dockerfile ${WORKSPACE}
          '''
        }
      }
      stage ('Publish Docker Image'){
        steps {
          sh '''
          docker push gcr.io/sylvan-bonbon-357404/helloworld
          '''
        }
      }
      stage ('Deploy to kubernetes'){
        steps{
          script {
            sh "kubectl config use-context gke_sylvan-bonbon-357404_us-central1-c_batch-14"
            sh "cd ${WORKSPACE}"
            sh "kubectl create -f '${WORKSPACE}'/k8s/deployment.yaml"
            sh "kubectl create -f '${WORKSPACE}'/k8s/service.yaml"
          }
         }
        }
     }
   }
      
