#!groovy

//@Library('slackNotifications-shared-library@master') _

pipeline {
	  agent any
   
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