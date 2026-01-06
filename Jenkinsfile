pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
        GITHUB_OWNER = 'sivaprasadpappala'
        GITHUB_REPO  = 'automerge'
        BASE_BRANCH  = 'main'
        FEATURE_BRANCH = "auto-pr-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BASE_BRANCH}",
                    url: "https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}.git"
            }
        }

        stage('Create Feature Branch & Commit') {
            steps {
                sh """
                git checkout -b ${FEATURE_BRANCH}
                echo "Automated change from Jenkins build ${BUILD_NUMBER}" >> auto.txt
                git add auto.txt
                git commit -m "Automated commit from Jenkins"
                """
            }
        }

        stage('Push Branch') {
            steps {
                sh """
                git push https://${GITHUB_TOKEN}@github.com/${GITHUB_OWNER}/${GITHUB_REPO}.git ${FEATURE_BRANCH}
                """
            }
        }

        stage('Create Pull Request') {
            steps {
                sh """
                curl -X POST https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPO}/pulls \\
                  -H "Authorization: token ${GITHUB_TOKEN}" \\
                  -H "Accept: application/vnd.github+json" \\
                  -d '{
                        "title": "Auto PR from Jenkins Build ${BUILD_NUMBER}",
                        "head": "${FEATURE_BRANCH}",
                        "base": "${BASE_BRANCH}",
                        "body": "This PR was automatically created by Jenkins."
                      }' > pr.json
                """
            }
        }

        stage('Extract PR Number') {
            steps {
                script {
                    env.PR_NUMBER = sh(
                        script: "jq -r .number pr.json",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Merge Pull Request') {
            steps {
                sh """
                curl -X PUT https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPO}/pulls/${PR_NUMBER}/merge \\
                  -H "Authorization: token ${GITHUB_TOKEN}" \\
                  -H "Accept: application/vnd.github+json" \\
                  -d '{
                        "merge_method": "squash"
                      }'
                """
            }
        }

        stage('Cleanup Branch') {
            steps {
                sh """
                curl -X DELETE https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPO}/git/refs/heads/${FEATURE_BRANCH} \\
                  -H "Authorization: token ${GITHUB_TOKEN}"
                """
            }
        }
    }
}
