#!groovy

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
	def TEST_LEVEL='%Testlevel%'
	def DEPLOYDEST='destructive'
	def workspace = WORKSPACE

    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://test.salesforce.com"


    def toolbelt = tool 'toolbelt'

    stage('Clean Workspace') 
	{
        try 
			{
            	deleteDir()
        	}
        catch (Exception e) 
			{
            	println('Unable to Clean WorkSpace.')
        	}
    }
    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') 
	{
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) 
	{	
	
	    	withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')])
			{
					// -------------------------------------------------------------------------
					// Authenticate to Salesforce using the server key.
					// -------------------------------------------------------------------------

				stage('Authorize to Salesforce') 
				{
					rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias SFDX"
		    		if (rc != 0) 
					{
						error 'Salesforce org authorization failed.'
		   			 }
				}


				// -------------------------------------------------------------------------
				// Install sfpowerkit.
				// -------------------------------------------------------------------------
				//stage('Install sfpowerkit')
				//{
					//bat 'sfpowerkit.bat'
				//}
				// -------------------------------------------------------------------------
				// Generate metadata.
				// -------------------------------------------------------------------------
				stage('Delta changes')
				{
				// -------------------------------------------------------------------------
    			// Delta and destructive changes.
    			// -------------------------------------------------------------------------
				bat 'sfdx sfpowerkit:project:diff --revisionfrom %PreviousCommitId% --revisionto %LatestCommitId% --output DeltaChanges'
	    		// -------------------------------------------------------------------------
    			// Delta and destructive changes.
    			// -------------------------------------------------------------------------
				if (Destructive_Changes=='true')
					{
						script
						{
							bat 'sfdx sfpowerkit:project:diff --revisionfrom %PreviousCommitId% --revisionto %LatestCommitId% --output DeltaChanges -x'
							bat 'copyDestructivefile.bat' 
						}
					}
				}
				
				stage('Deploy Destructive Changes') 
				{
					if (Deployment_Type=='Destructive Changes')
					{
						script
						{
							rc = command "${toolbelt}/sfdx force:mdapi:deploy -d destructive --wait 10 --targetusername SFDX"
							
		    			}
					 }
	    		}
           
	     		// -------------------------------------------------------------------------
	    		// Example shows how to run a check-only deploy.
	   			// -------------------------------------------------------------------------

				stage('Validate') 
				{
					if (Deployment_Type=='Validate')
					{
						script
						{
				
							if (TESTLEVEL=='NoTestRun') 
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --checkonly --wait 10 --targetusername SFDX "
							}
							else if (TESTLEVEL=='RunLocalTests') 
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --checkonly --wait 10 --targetusername SFDX --testlevel ${TESTLEVEL} --verbose --loglevel fatal"
							}
							else if (TESTLEVEL=='RunSpecifiedTests')
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --checkonly --wait 10 --targetusername SFDX --testlevel ${TESTLEVEL} -r %SpecifyTestClass% --verbose --loglevel fatal"
							}
   
							else (rc != 0) 
							{
								error 'Salesforce Validate step has failed.'
							}
		    			}
					 }
	    		}
            	// -------------------------------------------------------------------------
				// Deploy metadata and execute unit tests.
				// -------------------------------------------------------------------------
			    
				stage('Validate and Deploy') 
				{
					if (Deployment_Type=='Validate and Deploy')
					{
						script
						{
							if (TESTLEVEL=='NoTestRun') 
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --wait 10 --targetusername SFDX "
							}
							else if (TESTLEVEL=='RunLocalTests') 
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --wait 10 --targetusername SFDX --testlevel ${TESTLEVEL} --verbose --loglevel fatal"
							}
							else if (TESTLEVEL=='RunSpecifiedTests') 
							{
								println TESTLEVEL
								rc = command "${toolbelt}/sfdx force:source:deploy -p DeltaChanges --wait 10 --targetusername SFDX --testlevel ${TESTLEVEL} -r %SpecifyTestClass% --verbose --loglevel fatal"
							}
   
							else (rc != 0) 
							{
								error 'Salesforce Validate and Deploy step has failed.'
							}
		    			}
					}
	    		}
			}
	}
}

def command(script) 
{
    if (isUnix()) 
	{
        return sh(returnStatus: true, script: script);
    } else 
	{
		return bat(returnStatus: true, script: script);
    }
}
