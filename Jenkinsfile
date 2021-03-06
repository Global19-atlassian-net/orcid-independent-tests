def pytest(unit){
    try {
        startBrowser(unit)
        sh "export DISPLAY=:1.0 ; . .py_env/bin/activate && py.test -v -r fxE --junitxml results/${unit}.xml orcid/${unit}.py"
    } catch(Exception err) {
        def err_msg = err.getMessage()
        echo "PYTEST PROBLEM: $err_msg"
        throw err
    } finally {
        junit 'results/*.xml'
        sh 'rm -rf results/*.xml'
        stopBrowser(unit)
    }
}

def startBrowser(unit){
    echo "Creating xvfb..."
    sh "Xvfb :1 -screen 0 1024x758x16 -fbdir ${WORKSPACE}/xvfb & > /dev/null 2>&1 && echo \$! > /tmp/${unit}.pid"
    sh "cat /tmp/${unit}.pid"
}

def stopBrowser(unit){
    echo "Destroying xvfb..."
    sh "XVFB_PID=\$(cat /tmp/${unit}.pid) ; kill \$XVFB_PID" 
}

node {

    properties([
        buildDiscarder(
            logRotator(artifactDaysToKeepStr: '5', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')
        ),
        parameters([
            string(name: 'branch_to_build'       , defaultValue: 'master'                                         , description: 'Branch name to work on'),
            string(name: 'test_server'           , defaultValue: 'qa.orcid.org'                                   , description: 'Base domain name to test'),
            string(name: 'user_login'            , defaultValue: 'jeff2502test@mailinator.com'                    , description: 'Username'),
            string(name: 'user_pass'             , defaultValue: 'jeff2502test'                                   , description: 'Password'),
            string(name: 'orcid_id'              , defaultValue: '0000-0003-2390-3547'                            , description: 'Latest orcid ID'),
            string(name: 'search_value'          , defaultValue: 'jeff2502'                                       , description: 'Family name query format'),
            string(name: 'client_secrets_file'   , defaultValue: '/var/lib/jenkins/orcidclients.py'               , description: 'Properties file with predefined secrets')
        ]),
        [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false]
    ])

    git url: 'https://github.com/ORCID/orcid-independent-tests.git', branch: params.branch_to_build

    stage('Initialize'){
        sh "rm -f orcid/properties.py"
        writeFile file: 'testinputs.py', text: "test_server=\"$test_server\"\nsearchValue=\"$search_value\"\norcidId=\"$orcid_id\"\nuser_login=\"$user_login\"\nuser_pass=\"$user_pass\"\npassword=\"$user_pass\"\n"
        sh "cat $client_secrets_file testinputs.py > orcid/properties.py"
        sh "rm -rf .py_env results *.secret ${WORKSPACE}/xvfb && mkdir results ${WORKSPACE}/xvfb"
        sh "virtualenv .py_env"
        sh ". .py_env/bin/activate && pip2 install -q -r orcid/requirements.txt"
    }

    stage('TEST MEMBER 2.0 API POST UPDATE'){
        try {
            pytest 'test_member20_api_post_update'
        } catch(Exception err) {
            def err_msg = err.getMessage()
            echo "Tests problem: $err_msg"
        }
    }

    stage('TEST 2.0 API ALL ENDPOINTS') {
        try {
            pytest 'test_20api_all_endpoints'
        } catch(Exception err) {
            def err_msg = err.getMessage()
            echo "Tests problem: $err_msg"
        }
    }

    stage('TEST EXPECTED ERRORS'){
        try {
            pytest 'test_expected_errors'
        } catch(Exception err) {
            def err_msg = err.getMessage()
            echo "Tests problem: $err_msg"
        }
    }

    stage ('Finalize'){
        archive 'geckodriver.log'
        deleteDir()
    }

}

