#!groovy
import groovy.json.JsonSlurperClassic
node {

  def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    def toolbelt = tool 'toolbelt'

    stage('Checkout Source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
	
 
	
    	      stage('authorisatiom') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }
              }
        
        stage('Create Scratch Org') {
            // need to pull out assigned username
            rmsg = sh returnStdout: true, script: "\"${toolbelt}\" force:org:create -f config/project-scratch-def.json --json -s -a QAUbuntu"
            println(rmsg)
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            println(SFDC_USERNAME)
            robj = null
        }

    	stage('Set Default Scratch Org') {
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:config:set --global defaultusername=${SFDC_USERNAME} --json"
            if (rc != 0) { error 'Default scratch org failed' }
        }

        stage('Create password for scratch org') {
 			rmsg = sh returnStdout: true, script: "\"${toolbelt}\" force:user:password:generate --json"
			println(rmsg)
			def jsonSlurper = new JsonSlurperClassic()
			def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'password generation failed: ' + robj.message }
            robj = null
        }
	
        stage('Push To Scratch Org') {
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) { error 'Push failed'}	
            // assign permset
            rc = sh returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname purealoe"
            if (rc != 0) { error 'permset:assign failed'}
        }
}
