def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]    



pipeline {
    agent any
    environment{
        WORKSPACE = "${env.WORKSPACE}"
       
    }
    
     tools{
        maven 'localMaven'
        jdk 'localJdk'
     }
        


    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning the application code'
                git branch: 'main', url: 'https://github.com/NchangX/jenkins-CI-CD-Pipeline-Project.git'
                sh "mvn --version"
            }
        }
        
        stage('Build') {
            steps {
                sh 'java --version'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false               
                    }
                }
            } 
            
             stage('Unit Test'){
        steps {
            sh 'mvn test'
            }
        }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        
        stage ('SonarQube scanning'){
        steps {  
            withSonarQubeEnv('SonarQube') {
            sh """
                    mvn sonar:sonar \
          -Dsonar.projectKey=JavaWebApp \
          -Dsonar.host.url=http://172.31.14.115:9000 \
          -Dsonar.login=fc0b582e007eb897c2b83e8ab3f9936003d7d6a9
            
            """
            }
          }
        }
        
        stage("Quality Gate"){
    
          steps{
               
           waitForQualityGate abortPipeline: true
               
             }
                
            }
        stage("Upload articact to nexus"){
    
          steps{
               
           sh "mvn clean deploy -DskipTests"
               
             }
                
            }
        stage('Deploy to DEV env') {
              environment {
                HOSTS = "dev"
              }
              steps {
                  sh "ls"
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
              }
             }
             
             stage('Deploy to STAGE env') {
              environment {
                HOSTS = "stage"
              }
              steps {
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
              }
             }
             
             stage('Approval') {
                  steps {
                    input('Do you want to proceed?')
                  }
                }

             
             stage('Deploy to Prod env') {
              environment {
                HOSTS = "prod"
              }
              steps {
                sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
              }
             }
    }
             post {
                always {

            echo 'I will always say Hello again!'
            slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"

                }
            }
}
