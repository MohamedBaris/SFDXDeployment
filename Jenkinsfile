node {
    // Define environment variables
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR='src'
    def TEST_LEVEL='NoTestRun'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"

    // Define Salesforce CLI tool
    def toolbelt = tool 'toolbelt'

    // Debug: Print out environment variables and toolbelt path
    echo "SF_CONSUMER_KEY: ${SF_CONSUMER_KEY}"
    echo "SF_USERNAME: ${SF_USERNAME}"
    echo "SERVER_KEY_CREDENTIALS_ID: ${SERVER_KEY_CREDENTIALS_ID}"
    echo "SF_INSTANCE_URL: ${SF_INSTANCE_URL}"
    echo "toolbelt path: ${toolbelt}"

    // Checkout source code
    stage('Checkout source') {
        checkout scm
    }

    // Run all stages with access to the Salesforce JWT key credentials
    withEnv(["HOME=${env.WORKSPACE}"]) {  
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {

            // Authenticate to Salesforce using the server key
            stage('Authorize to Salesforce') {
                // Install required plugin (if not already installed)
                command "${toolbelt}/sfdx plugins:install sfdx-git-delta@v5.38.2"
                // Debug: Check installed plugins
                command "${toolbelt}/sfdx plugins"
                // Authenticate to Salesforce
                command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias UAT"
            }

            // Download the source delta
            stage('Download the difference') {
                command "git fetch"
                command "git fetch --unshallow"
                // Execute source delta command
                command "${toolbelt}/sfdx sgd:source:delta --to Feature1 --from main --output ."
            }

            // Deploy to Salesforce
            stage('Deploy to Salesforce') {
                // Deploy source code
                command "${toolbelt}/sfdx force:source:deploy -x package/package.xml --postdestructivechanges destructiveChanges/destructiveChanges.xml --targetusername UAT"
            }
        }
    }
}

// Function to execute commands based on OS
def command(script) {
    if (isUnix()) {
        return sh(script);
    } else {
        return bat(script);
    }
}
