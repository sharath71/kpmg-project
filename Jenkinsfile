pipeline {
  agent any
    
  tools {nodejs "node"}
    
  stages {
        
    stage('Git') {
      steps {
        git branch: 'main', credentialsId: '<git_sharath_id>', url: 'https://github.com/sharath71/kpmg.git'
      }
    }
     
    stage('Build') {
      steps {
        sh 'npm install'
         sh 'npm run dev'
      }
    }  
    stage('Checkmarx Scan') {
            steps {
                script {
                    // Define Checkmarx scan parameters
                    def checkmarxScanParameters = [
                        checkmarxServer: 'CheckmarxServerName', // Name of the Checkmarx server configured in Jenkins
                        credentialsId: '<CHECKMARX_CREDENTIALS>', // Jenkins credentials ID with Checkmarx credentials
                        projectName: 'YourProjectName', // The name of your Checkmarx project
                        sourceLocation: 'SourceControl', // Source control system (e.g., Git)
                        sourceOrigin: 'jenkins',
                        engineConfigurationName: 'Default Configuration',
                        presetName: 'HighlySensistiveData',
                        folderExclusions: 'node_modules',
                        incremental: false,
                        comment: 'Jenkins Checkmarx scan'
                    ]

                    // Trigger the Checkmarx scan
                    def scanResult = checkmarxScan(parameters: checkmarxScanParameters)
                }
            }
        }

        stage('Publish Checkmarx Results') {
            steps {
                // Publish Checkmarx scan results
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: 'CheckmarxReports',
                    reportFiles: 'CxXMLResults*.xml',
                    reportName: 'Checkmarx Scan Results',
                    reportTitles: ''
                ])
            }
        }
 stage('Docker build') {
            steps {
                sh 'docker build -t sharath71/kpmg:0.0.1 .'
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'withcredential', passwordVariable: 'hubPwd', usernameVariable: 'username')]) {
                    sh "docker login -u ${username} -p ${hubPwd}"
                    sh "docker push sharath71/kpmg:0.0.1"
                }
                
            }
    stage('Deploy to ArgoCD') {
            steps {
                script {
                    def argocdServer = 'ARGOCD_SERVER_URL'
                    def argocdToken = credentials('ARGOCD_TOKEN')

                    def projectName = 'my-project'
                    def applicationName = 'my-application'

                    def argocdCLI = docker.image('sharath71/kpmg:0.0.1').inside {
                        sh 'argocd login $argocdServer --insecure --username admin --password $argocdToken'
                        sh "argocd app set $applicationName --project $projectName"
                        sh "argocd app sync $applicationName"
                    }
                }
            }
            
            
