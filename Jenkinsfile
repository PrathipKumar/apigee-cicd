#!groovy

@Library('slackNotifications-shared-library@master') _

pipeline {
	  agent any
    environment {
        //getting the current stable/deployed revision...this is used in undeloy.sh in case of failure...
        stable_revision = sh(script: 'curl -H "Authorization: Basic $base64encoded" "https://api.enterprise.apigee.com/v1/organizations/kprathip-eval/apis/HR-API/deployments"', returnStdout: true).trim()
    } 

    stages {
        stage('Initial-Checks') {
            steps {     
				sendNotifications 'STARTED the build'
                bat "npm -v"
                bat "mvn -v"
                echo "$apigeeUsername"  				
        }}
		stage('Policy-Code Analysis') {
            steps {
                bat "npm install -g apigeelint"
                bat "apigeelint -s HR-API/apiproxy/ -f html.js > $WORKSPACE/lint-report.html"		
				bat "LINT_STATUS = sh(script: '$?', returnStatus: true)"
				script{
					if(LINT_STATUS == 0)
						echo $LINT_STATUS	
				}
							
				publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '.', reportFiles: 'lint-report.html', reportName: 'HTML Report', reportTitles: ''])						
            }
        }
		stage('Unit-Test-With-Coverage') {
            steps {
                script {
                    try {
                        bat "npm install"
                        bat "npm test test/unit/*.js"
                        bat "npm run coverage test/unit/*.js"
                    } catch (e) {
                        throw e
                    } finally {
                        bat "cd coverage && cp cobertura-coverage.xml $WORKSPACE"
                        step([$class: 'CoberturaPublisher', coberturaReportFile: 'cobertura-coverage.xml'])
                    }
                }
            }
        }
		stage('Deploy to Production') {
            steps {
                 //deploy using maven plugin
                 
                 // deploy only proxy and deploy both proxy and config based on edge.js update
                bat "sh && sh deploy.sh"
                //bat "mvn -f HR-API/pom.xml install -Pprod -Dusername=${apigeeUsername} -Dpassword=${apigeePassword} -Dapigee.config.options=update"
            }
        }
		stage('Integration Tests') {
            steps {
                script {
                    try {
                        // using credentials.sh to get the client_id and secret of the app..
                        // thought of using them in cucumber oauth feature
                         bat "sh && sh credentials.sh"
                        bat "cd $WORKSPACE/test/integration && npm install"
                        bat "cd $WORKSPACE/test/integration && npm test"
                    } catch (e) {
						sendNotifications 'Integration test failed'
						error("Pipeline stopped")
                        //if tests fail, I have used an shell script which has 3 APIs to undeploy, delete current revision & deploy previous stable prevision
                        //bat "sh && sh undeploy.sh"
                        //throw e
                    } finally {
                        // generate cucumber reports in both Test Pass/Fail scenario
                        bat "cd $WORKSPACE/test/integration && cp reports.json $WORKSPACE"
                        cucumber fileIncludePattern: 'reports.json'
                        //build job: 'cucumber-report'
                    }
                }
            }
        }		
 
    }
	post {
        always {
             cucumberSlackSend channel: 'devops', json: '$WORKSPACE/reports.json'
             sendNotifications currentBuild.result
        }
    }

 }