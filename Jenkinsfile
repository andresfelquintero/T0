import groovy.json.JsonOutput
// Add whichever params you think you'd most want to have


def newJiraIssue(summary, description){
    def testIssue = [fields: [ project: [key: 'CMSTZDEV'],
                       summary: summary,
                       description: description,
                       issuetype: [name: 'Task']]]  
                       
    response = jiraNewIssue issue: testIssue, site: 'CMSTZ'
    // echo response.successful.toString()
    // echo response.data.toString()
    JIRA_ISSUE_ID = response.data.id
    assignResponse = jiraAssignIssue site: 'CMSTZ', idOrKey: JIRA_ISSUE_ID, userName: 'vjankaus'
    // JIRA_ISSUE_ID = 'CMSTZDEV-153'
    println(JIRA_ISSUE_ID)
}

def getProject(){
    def issues = jiraJqlSearch jql: 'PROJECT = CMSTZDEV', site: 'CMSTZ', failOnError: true
    // echo issues.data.toString()
}

def addJiraComment(text){
    response = jiraAddComment site: 'CMSTZ', idOrKey: JIRA_ISSUE_ID, comment: text
    echo response.data.toString()
}

node('t0ReplayNode') {

    environment {
        JIRA_ISSUE_ID = ""
        SHELL_OUTPUT = ""
        GIT_COMMIT_MSG = ""
        GIT_DIR=""
        DESCRIPTION = "Add more info about the replay later."
    }
    stage('CleanupBefore') {
        checkout scm

        sh '''
            # git checkout ghprbActualCommit
            echo 'Starting a cleanup before the replay.'
            pwd
            #switch to working dir
            cd /data/tier0/
            #stop the agent
            ./00_stop_agent.sh
            # remove all the previous jobs from condor q
            #sleep 10
            condor_rm -all
            sleep 10
        '''
        script{
            GIT_COMMIT_MSG= sh(returnStdout: true, script: "git log --oneline -1").trim()
            echo GIT_COMMIT_MSG
            newJiraIssue(GIT_COMMIT_MSG, "Jenkins description")
            echo JIRA_ISSUE_ID
            echo 'CleanupBefore'
            // addJiraComment('CleanupBefore')

        }
    }
    stage('DeployTheAgent') {
        sh '''
            gitdir=$(pwd)
            echo ${gitdir}
            echo 'deploy the new agent+T0 config.'
            cd /data/tier0/
        
            ./00_software.sh
            #sleep 30
            #deploy them
            ./00_deploy_replay.sh
            #sleep 60 
            echo 'deployment done.'
            #TODO later, refactor this ugly workaround to be deployed together with T0 repo
            cp ${gitdir}/etc/ReplayOfflineConfiguration.py /data/tier0/admin/ReplayOfflineConfiguration.py
            #here we should have some sanity check? or after the agent is started
        '''
        script{
            echo 'Deploy agent'
            // addJiraComment('Deploy agent')
            SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/compile.py /data/tier0/admin/ReplayOfflineConfiguration.py')
            addJiraComment(SHELL_OUTPUT)
            SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/compile.py /data/tier0/admin/ReplayOfflineConfiguration.py 1')
            addJiraComment('DeployAgent done')
        }
    }
    stage('StartTheAgent') {
        sh '''
        echo 'Start the agent.'
        cd /data/tier0/
        #start the agent
        echo 'passing start_agent'
        ./00_start_agent.sh
        #run couch processes for a while
        sleep 150
        '''
        script{
            echo JIRA_ISSUE_ID
            echo 'Start agent'
            // addJiraComment('Start agent')

        }
  
    }
    stage('ReplayChecks') {
        parallel (
            'PauseProgress': 
                { stage('PauseProgress'){
                    script{
                        echo 'passing Checking the Pause status.'
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/compile.py /data/tier0/jenkins/replayWorkflowStatus.py')
                        addJiraComment(SHELL_OUTPUT)
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/compile.py /data/tier0/jenkins/replayWorkflowStatus.py 1')
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/replayWorkflowStatus.py Paused')
                        echo 'passing Checking the Pause status.'
                        echo SHELL_OUTPUT
                        addJiraComment(SHELL_OUTPUT)
                        addJiraComment('PauseProgress done')
       
                    }
                }},
            'RepackProgress': 
                { stage('RepackProgress'){
                    script{
                        
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/replayWorkflowStatus.py Repack')
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/replayWorkflowStatus.py Repack')
                        echo 'passing Checking the Repack status.'
                        echo SHELL_OUTPUT
                        addJiraComment(SHELL_OUTPUT)
                        addJiraComment('RepackProgress done')
       
                    }
                }},
            'ExpressProgress': { 
                stage('ExpressProgress'){
                    script{
                        
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/replayWorkflowStatus.py Express')
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/replayWorkflowStatus.py Express')
                        echo 'passing Checking the Express status.'
                        echo SHELL_OUTPUT
                        addJiraComment(SHELL_OUTPUT)
                        addJiraComment('ExpressProgress done')
                    }
                }
            },
            'FilesetProgress': {
                stage('FilesetProgress'){
                    script{
                        
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/replayWorkflowStatus.py Fileset')
                        SHELL_OUTPUT = sh(returnStdout: true, script: 'python /data/tier0/jenkins/message.py /data/tier0/jenkins/replayWorkflowStatus.py Fileset')
                        echo 'passing Checking the Filesets status.'
                        echo SHELL_OUTPUT
                        addJiraComment(SHELL_OUTPUT)
                        addJiraComment('FilesetProgress done')
                    
                    }
                }                    
            }
        )
        echo "end of script"
        addJiraComment('The replay finished successfully.')

    }
}
