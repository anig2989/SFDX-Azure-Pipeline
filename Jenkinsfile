#!groovy

node {
    
    // -------------------------------------------------------------------------
    // Defining Org Variables
    // -------------------------------------------------------------------------
	
    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR='force-app'
    def SF_DELTA_FOLDER='DELTA_PKG'
    def TEST_LEVEL= env.TEST_LEVEL
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"
	
    def DEPLOYMENT_TYPE=env.DEPLOYMENT_TYPE // Incremental Deployment = DELTA ; Full Deployment = FULL
    def SF_SOURCE_COMMIT_ID=env.SOURCE_BRANCH
    def SF_TARGET_COMMIT_ID=env.TARGET_BRANCH
    
    //Defining SFDX took kit path against toolbelt
    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }
	


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials
    // -------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) {	
	echo "workspace directory is ${workspace}"
	    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {
		// -------------------------------------------------------------------------
		// Authenticate to Salesforce using the server key.
		// -------------------------------------------------------------------------

		stage('Install Powerkit Plugin') {
        		rc = sh "echo 'y' | ${toolbelt}sfdx plugins:install sfpowerkit"
    		}
		    
		stage('Authorize to Salesforce') {
      			def rc = sh '${toolbelt}sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias UAT'
			//echo "Authorization Sussessful."
			if (rc != 0) {
    				error 'Authorization Failed.'
			}
		}
		 
		// -------------------------------------------------------------------------
		// Creating Delta Package with the changes.
		// -------------------------------------------------------------------------

		stage('Create_Delta_Package') {
      			if (DEPLOYMENT_TYPE == 'DELTA'){
				echo "Deploying DELTA Components from Repository"
            			rc = sh "${toolbelt}sfdx sfpowerkit:project:diff -d ${SF_DELTA_FOLDER} -r ${SF_SOURCE_COMMIT_ID} -t ${SF_TARGET_COMMIT_ID}"
          		}
          		else{
              			echo "Deploying All Components from Repository"
          		}
		}

		// -------------------------------------------------------------------------
		// Validating Stage.
		// -------------------------------------------------------------------------

		stage('Package_Validation') {
      			if (DEPLOYMENT_TYPE == 'DELTA')
            		{
            			rc = sh "${toolbelt}sfdx force:source:deploy -c -p ${SF_DELTA_FOLDER}/${DEPLOYDIR} -u ${SF_USERNAME} -w 500 -l ${TEST_LEVEL}"
				echo "Component Validation Successful"
            		}
            		else
            		{
            			rc = sh "${toolbelt}sfdx force:source:deploy -c -p ${DEPLOYDIR} -u ${SF_USERNAME} -w 500 -l ${TEST_LEVEL}"
				echo "Component Validation Successful"
            		}
        	}
		    
		// -------------------------------------------------------------------------
		// Deployment Stage.
		// -------------------------------------------------------------------------
    
		    
		stage('Package_Deployment') {
      			if (DEPLOYMENT_TYPE == 'DELTA')
            		{
            			rc = sh "${toolbelt}sfdx force:source:deploy -p ${SF_DELTA_FOLDER}/${DEPLOYDIR} -u ${SF_USERNAME} -w 500 -l ${TEST_LEVEL}"
				echo "Component Validation Successful"
            		}
            		else
            		{
            			rc = sh "${toolbelt}sfdx force:source:deploy -p ${DEPLOYDIR} -u ${SF_USERNAME} -w 500 -l ${TEST_LEVEL}"
            			echo "Component Validation Successful"
            		}
        	}
		    
		// -------------------------------------------------------------------------
		// Deploy metadata and execute unit tests.
		// -------------------------------------------------------------------------

		//stage('Deploy and Run Tests') {
		//    rc = command "${toolbelt}/sfdx force:mdapi:deploy --wait 10 --deploydir ${DEPLOYDIR} --targetusername UAT --testlevel ${TEST_LEVEL}"
		//    if (rc != 0) {
		//	error 'Salesforce deploy and test run failed.'
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
