#!groovy
import groovy.json.JsonSlurperClassic
node {
    environment {
        SFDX_USE_GENERIC_UNIX_KEYCHAIN = true
    }
    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    def toolbelt = '/usr/local/bin'

    set_github_commit_status() {
        echo "Settings GitHub Commit $GIT_COMMIT to the status $1..."
        curl "https://api.github.com/repos/$GITHUB_ORG/$GITHUB_REPO/statuses/$GIT_COMMIT?access_token=$GITHUB_ACCESS_TOKEN" \
        -H "Content-Type: application/json" \
        -X POST \
        -d "{\"state\": \"$1\", \"description\": \"$2\", \"target_url\": \"$BUILD_URL/console\", \"context\": \"continuous-integration/jenkins/push\"}" \
        -s > /dev/null #Hide curl output
    }

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
        set_github_commit_status 'pending' 'The build is processing...'
    }
    
    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
            rmsg = sh returnStdout: true, script: "${toolbelt}/sfdx force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
            printf rmsg
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null

        }

        stage('Push To Test Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
        }

        stage('Run Apex Test') {
            sh "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --wait --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    set_github_commit_status 'failure' 'Unit tests checks failed'
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
            set_github_commit_status 'success' 'The build is valid'
        }
    }
}
