pipeline {
  agent any

  environment {
    GITHUB_OWNER = "sivaprasadpappala"
    GITHUB_REPO  = "automerge"
    BASE_BRANCH  = "main"
    FEATURE_BRANCH = "auto-pr-${BUILD_NUMBER}"
    GITHUB_API = "https://api.github.com"
  }

  stages {

    stage("Checkout") {
      steps {
        git branch: "${BASE_BRANCH}",
            url: "https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}.git"
      }
    }

    stage("Create Branch & Commit") {
      steps {
        sh '''
          git config user.name "Jenkins CI"
          git config user.email "jenkins@ci.local"

          git checkout -b ${FEATURE_BRANCH}
          echo "Automated PR created at $(date)" >> auto-pr.txt
          git add auto-pr.txt
          git commit -m "Automated commit from Jenkins"
        '''
      }
    }

    stage("Push Branch") {
      steps {
        withCredentials([string(
          credentialsId: 'github-token',
          variable: 'GITHUB_TOKEN'
        )]) {
          sh '''
            git push https://${GITHUB_TOKEN}@github.com/${GITHUB_OWNER}/${GITHUB_REPO}.git ${FEATURE_BRANCH}
          '''
        }
      }
    }

    stage("Create Pull Request") {
      steps {
        withCredentials([string(
          credentialsId: 'github-token',
          variable: 'GITHUB_TOKEN'
        )]) {
          sh '''
            RESPONSE=$(curl -s -w "\\n%{http_code}" -X POST \
              -H "Authorization: Bearer ${GITHUB_TOKEN}" \
              -H "Accept: application/vnd.github+json" \
              ${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/pulls \
              -d '{
                "title": "Automated PR from Jenkins",
                "head": "'"${FEATURE_BRANCH}"'",
                "base": "'"${BASE_BRANCH}"'"
              }')

            BODY=$(echo "$RESPONSE" | head -n -1)
            STATUS=$(echo "$RESPONSE" | tail -n 1)

            echo "HTTP Status: $STATUS"
            echo "$BODY" | jq .

            [ "$STATUS" -eq 201 ] || exit 1

            echo "$BODY" > pr.json
          '''
        }
      }
    }

    stage("Merge Pull Request") {
      steps {
        withCredentials([string(
          credentialsId: 'github-token',
          variable: 'GITHUB_TOKEN'
        )]) {
          sh '''
            PR_NUMBER=$(jq -r '.number' pr.json)

            echo "Merging PR #${PR_NUMBER}"

            curl -s -X PUT \
              -H "Authorization: Bearer ${GITHUB_TOKEN}" \
              -H "Accept: application/vnd.github+json" \
              ${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/pulls/${PR_NUMBER}/merge
          '''
        }
      }
    }
  }

  post {
    success {
      echo "PR created and merged successfully"
    }
    failure {
      echo "Pipeline failed"
    }
  }
}
