pipeline {
    agent any
    stages {
        stage('Run Terrascan') {
            steps {
                script {
                    // Run Terrascan and capture output and status
                    def scanStatus = sh(
                        script: '''
                        docker run --rm -v /var/lib/jenkins/workspace/checkov_terraform:/iac accurics/terrascan:latest scan -d /iac/EKS_Terraform -o json > terrascan_output.json
                        ''',
                        returnStatus: true
                    )
                    
                    // Archive Terrascan results for analysis
                    archiveArtifacts artifacts: 'terrascan_output.json', allowEmptyArchive: true

                    // Parse JSON Output to Decide Pass or Fail
                    def output = readJSON file: 'terrascan_output.json'
                    def violatedPolicies = output.results.policies.validated.size()

                    // Fail the pipeline only if violated policies > 0
                    if (violatedPolicies > 0) {
                        error("Terrascan found vulnerabilities. Check terrascan_output.json for details.")
                    } else {
                        echo "No vulnerabilities found. Terrascan passed successfully."
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed due to detected vulnerabilities.'
        }
    }
}
