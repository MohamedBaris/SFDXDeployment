#!groovy

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR='src'
    def TEST_LEVEL='NoTestRun'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"


    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

	environment {

    PATH = "C:\\WINDOWS\\SYSTEM32"

}



		
    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) {	
	
	    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {
		// -------------------------------------------------------------------------
		// Authenticate to Salesforce using the server key.
		// -------------------------------------------------------------------------

		stage('Authorize to Salesforce') {
				rc = command "echo y | sfdx plugins:install sfdx-git-delta"
			rc = command "echo %PATH%"
			rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias UAT"
		    if (rc != 0) {
			error 'Salesforce org authorization failed.'
		    }
		}


		// -------------------------------------------------------------------------
		// Deploy metadata and execute unit tests.
		// -------------------------------------------------------------------------

		stage('Download the Main') {
			  rc = command "git checkout Feature1"
		    rc = command "git checkout main"
		//	rc = command "${toolbelt}/sfdx force:source:deploy -x ${DEPLOYDIR} --targetusername UAT"
		    if (rc != 0) {
			error 'Salesforce deploy and test run failed.'
		    }
		}

		
		stage('Download the difference') {
			rc = command "git fetch"
			rc = command "git fetch --unshallow"
		    rc = command "${toolbelt}/sfdx sgd:source:delta --to Feature1 --from main --output ."
		//	rc = command "${toolbelt}/sfdx force:source:deploy -x ${DEPLOYDIR} --targetusername UAT"
		    if (rc != 0) {
			error 'Salesforce deploy and test run failed.'
		    }
		}


		stage('Deploy to Salesforce') {
		    rc = command "${toolbelt}/sfdx force:source:deploy -x package/package.xml --postdestructivechanges destructiveChanges/destructiveChanges.xml --targetusername UAT"
		//	rc = command "${toolbelt}/sfdx force:source:deploy -x ${DEPLOYDIR} --targetusername UAT"
		    if (rc != 0) {
			error 'Salesforce deploy and test run failed.'
		    }
		}

		// -------------------------------------------------------------------------
		// Example shows how to run a check-only deploy.
		// -------------------------------------------------------------------------

		//stage('Check Only Deploy') {
		//    rc = command "${toolbelt}/sfdx force:mdapi:deploy --checkonly --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"
		//    if (rc != 0) {
		//        error 'Salesforce deploy failed.'
		//    }
		//}
	    }
	}
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}
