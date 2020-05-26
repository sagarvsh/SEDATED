properties([
    parameters([
        text(defaultValue: '1234567890example1234567890commit1234567', description: 'Enter the 40 char commit id(s) with space of new line as delimeter. Example: 1234567890example1234567890commit1234567', name: 'CommitIDs', trim: true),
        choice(choices: ['False Positive', 'I acknowledge its True Positive, it will be remediated soon'], description: '', name: 'Reason')
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
            wrap([$class: 'BuildUser']){
                sh """#!/bin/sh
                git clone git@github.com:sagarvsh/SEDATED.git
                cd sedated/config/whitelists
                git pull
                echo "$validCommits" >> commit_whitelist.txt
                echo "| $BUILD_ID | $TIMESTAMP | $validCommits2 | $Reason | $BUILD_USER |" >> audit_commit_whitelist.md
                cat audit_commit_whitelist.md
                git commit -am "commit $validCommits2"
                git push origin master
                """
            }
        }

        /* stage('Record event in GitHub Issue'){
            wrap([$class: 'BuildUser']){
                sh "echo '{\"title\":\"[auto-creation] Commit ID(s) whitelisted by '$BUILD_USER' \",\"body\":\" @'BUILD_USER_ID' whitelisted '$validCommits2'\",\"assignee\":[\"sagarvsh\"],\"labels\":[\"auto-creation\"]}' > issue.json"

                sh '''
                response=$(curl -X POST -H "Authorization: token $githubtoken" https://github.com/api/v3/repos/svalage/sedated/issues --data @issue.json)
                issuenumber=$(echo $response | jq '.number')
                curl -X PATCH -H "Authorization: token $githubtoken" https://github.com/api/v3/repos/svalage/sedated/issues/$issuenumber --data '{"stage":"closed"}'
                '''
            }
        } */
        cleanWs()
}
