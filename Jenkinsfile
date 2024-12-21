pipeline {
    agent any
    stages {
        stage('Run Terrascan') {
            steps {
                script {
                    // Run Terrascan and save the JSON output
                    def scanStatus = sh(
                        script: '''
                        docker run --rm -v /var/lib/jenkins/workspace/checkov_terraform:/iac accurics/terrascan:latest scan -d /iac/EKS_Terraform -o json > terrascan_output.json
                        ''',
                        returnStatus: true
                    )
                    
                    // Archive Terrascan results
                    archiveArtifacts artifacts: 'terrascan_output.json', allowEmptyArchive: true
                    
                    // Parse JSON output manually using Groovy
                    def jsonContent = readFile('terrascan_output.json')
                    def parsedJSON = new groovy.json.JsonSlurper().parseText(jsonContent)
                    
                    // Get the count of violated policies
                    def violatedPolicies = parsedJSON.results.policies.findAll { it.violated }.size()

                    // Fail pipeline if there are violations
                    if (violatedPolicies > 0) {
                        error("Terrascan found ${violatedPolicies} vulnerabilities. Check terrascan_output.json for details.")
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
