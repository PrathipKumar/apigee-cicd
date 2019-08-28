#!groovy

//@Library('slackNotifications-shared-library@master') _

pipeline {
	  agent any
    environment {
        //getting the current stable/deployed revision...this is used in undeloy.sh in case of failure...
        stable_revision = sh(script: 'curl -H "Authorization: Basic $base64encoded" "https://api.enterprise.apigee.com/v1/organizations/kprathip-eval/apis/HR-API/deployments"', returnStdout: true).trim()
    } 

    stages {
        stage('Initial-Checks') {
            steps {                
                bat "npm -v"
                bat "mvn -v"
                echo "$apigeeUsername"                
        }}
		stage('Policy-Code Analysis') {
            steps {
                bat "npm install -g apigeelint"
                bat "apigeelint -s HR-API/apiproxy/ -f codeframe.js"
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
                //bat "sh && sh deploy.sh"
                bat "mvn -f HR-API/pom.xml install -Pprod -Dusername=${apigeeUsername} -Dpassword=${apigeePassword} -Dapigee.config.options=update"
            }
        }		
 
    }

 }