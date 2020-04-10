#!groovy

import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def SF_USERNAME=env.HUB_ORG_DH
    def SERVER_KEY_CREDENTIALS_ID=env.JWT_CRED_ID_DH
    def SF_INSTANCE_URL = env.SFDC_HOST_DH ?: "https://login.salesforce.com"

    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

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
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                //Force logout to avoid ERROR running force:org:create:  ENOENT: no such file or directory, open 'C:\Program Files (x86)\Jenkins\workspace\Jenkins_Webhook_master@tmp\secretFiles\1fdeef11-05b5-446b-b41b-8818739303b3\server.key'
                rb = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:logout --targetusername  ${SF_USERNAME} --noprompt"
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile \"${server_key_file}\" --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Query Scratch Org Limits.
            // -------------------------------------------------------------------------
            stage('Query Scratch Org Limits') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:limits:api:display --targetusername ${SF_USERNAME}"
                if (rc != 0) {
                    error 'Salesforce query scratch org limits failed.'
                }
            }
        }
    }
}
