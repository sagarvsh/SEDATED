properties([
    parameters([
        text(defaultValue: '1234567890example1234567890commit1234567', description: 'Enter the 40 char commit id(s) with space of new line as delimeter. Example: 1234567890example1234567890commit1234567', name: 'CommitIDs', trim: true),
        choice(choices: ['False Positive', 'Fixed', 'I acknowledge its True Positive, it will be remediated soon'], description: '', name: 'Reason')
    ])
])

node {
    def validCommits = ""
    def validCommits2 = ""
    def invalidCommits = ""
    def TIMESTAMP = new Date()
    // TIMESTAMP=`date +%Y-%m-%d_%H-%M-%S`

        stage('Validate') {
            def chkCommit = "${params.CommitIDs}"
            def listOfCommits = "${params.CommitIDs}".split()
            if (!chkCommit) {
                error("Commit List is empty")
            }
            for (commit in listOfCommits) {
                echo 'validating ' + commit
                if (commit !=null && commit.length()!=40) {
                    invalidCommits += commit + "\n"
                    echo commit + ' not valid'
                }
                else {
                    validCommits += commit + "\n"
                    validCommits2 += commit + " "
                    echo commit + ' is valid'
                }
            }
            if (invalidCommits != null && !invalidCommits.isEmpty()) {
                echo invalidCommits + 'Above Commit ID(s) are not valid, correct them and rerun'
                error("One or more of your Commit ID(s) are not valid")
            }
            validCommits = validCommits.substring(0, validCommits.length() - 1)
            validCommits2 = validCommits2.substring(0, validCommits2.length() - 1)
            echo "${params.CommitIDs}"
            echo validCommits
            echo validCommits2
        }

        stage('Whitelist') {
            sshagent(['GitHubSSH']) {
            wrap([$class: 'BuildUser']){
                sh """#!/bin/sh
                git clone git@github.com:sagarvsh/sedated.git
                cd sedated/config/whitelists
                git checkout -b develop
                git pull origin develop
                echo "$validCommits" >> commit_whitelist.txt
                echo "| $BUILD_ID | $TIMESTAMP | $validCommits2 | $Reason | $BUILD_USER |" >> audit_commit_whitelist.md
                cat audit_commit_whitelist.md
                git commit -am "commit $validCommits2"
                git push origin develop
                """
            }
            }
        }

        stage('Record event in GitHub Issue'){
            // properties([[$class: 'GithubProjectProperty', projectUrlStr: 'https://github.com/api/v3/repos/sagarvsh/sedated.git']]) {
            //wrap([$class: 'GithubProjectProperty']) {
            withCredentials([string(credentialsId: 'k8sgithubmac', variable: 'TOKEN')]) {
            wrap([$class: 'BuildUser']){
                sh "echo '{\"title\":\"[auto-creation] SEDATED - '$BUILD_USER_FIRST_NAME' whitelisted Commit ID(s)\",\"body\":\" @'$BUILD_USER_ID' Hi '$BUILD_USER_FIRST_NAME', Thank you for using self service utility to whitelist commit id(s). Following commit id(s) are whitelisted '$validCommits2'\",\"assignees\":[\"sagarvsh\"],\"labels\":[\"auto-creation\"]}' > issue.json"

                sh '''
                response=$(curl -X POST -H "Authorization: token $TOKEN" https://api.github.com/repos/sagarvsh/sedated/issues --data @issue.json)
                << jqNotInstalled
                issuenumber=$(echo $response | jq '.number')
                jqNotInstalled
                << AutoClose
                curl -X PATCH -H "Authorization: token $TOKEN" https://api.github.com/repos/sagarvsh/sedated/issues/$issuenumber --data '{"state":"closed"}'
                AutoClose
                '''
            }
            }
        } 
        cleanWs()
}
