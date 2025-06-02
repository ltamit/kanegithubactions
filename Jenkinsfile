pipeline {
  agent any

  parameters {
    string(name: 'TEST_RUN_ID', description: 'LambdaTest Test Run ID')
    string(name: 'REGION', description: 'Region (e.g. eastus, centralindia)')
    string(name: 'TITLE', description: 'Unique Build Title')
  }

  environment {
    AUTH_HEADER = credentials('LT_AUTH_BASE64') // Jenkins credential with ID 'LT_AUTH_BASE64'
  }

  stages {
    stage('Trigger HyperExecute Job') {
      steps {
        script {
          def payload = """{
            "test_run_id": "${params.TEST_RUN_ID}",
            "concurrency": 1,
            "title": "${params.TITLE}",
            "region": "${params.REGION}"
          }"""

          def triggerResponse = sh(
            script: """curl -s --location 'https://test-manager-api.lambdatest.com/api/atm/v1/hyperexecute' \
              --header 'Content-Type: application/json' \
              --header 'Authorization: Basic ${AUTH_HEADER}' \
              --data '${payload}'""",
            returnStdout: true
          ).trim()

          echo "Trigger Response: ${triggerResponse}"

          def json = readJSON text: triggerResponse
          env.APP_JOB_ID = json.app_job_id ?: ''
          if (!env.APP_JOB_ID?.trim()) {
            error "❌ app_job_id not found. Exiting."
          }
          echo "✅ App Job ID: ${env.APP_JOB_ID}"
        }
      }
    }

    stage('Wait for 5 Minutes') {
      steps {
        echo "⏳ Waiting for 5 minutes before checking job status..."
        sleep time: 5, unit: 'MINUTES'
      }
    }

    stage('Fetch Job Status') {
      steps {
        script {
          def statusResponse = sh(
            script: """curl -s -X GET "https://api.hyperexecute.cloud/v2.0/job/${env.APP_JOB_ID}" \
              -H "accept: application/json" \
              -H "Authorization: Basic ${AUTH_HEADER}" """,
            returnStdout: true
          ).trim()

          echo "Status Response: ${statusResponse}"

          try {
            def statusJson = readJSON text: statusResponse
            def jobStatus = statusJson.data?.status ?: 'UNKNOWN'
            echo "✅ Job Status: ${jobStatus}"
          } catch (Exception e) {
            error "❌ Failed to parse job status JSON: ${e.message}"
          }
        }
      }
    }
  }
}
