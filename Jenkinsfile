#!groovy

//@Library('slackNotifications-shared-library@master') _

pipeline {
	  agent any
    environment {
        //getting the current stable/deployed revision...this is used in undeloy.sh in case of failure...
        stable_revision = sh(script: 'curl -H "Authorization: Basic $base64encoded" "https://api.enterprise.apigee.com/v1/organizations/kprathip-eval/apis/HR-API/deployments" | jq -r ".environment[0].revision[0].name"', returnStdout: true).trim()
    } 

    stages {
        stage('Initial-Checks') {
            steps {                
                bat "npm -v"
                bat "mvn -v"
                echo "$apigeeUsername"
                echo "Stable Revision: ${env.stable_revision}"
        }}  
 
    }

 }