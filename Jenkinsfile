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
                    
                    // Get the number of medium and high-severity violations
                    def mediumViolations = parsedJSON.results.violations.findAll { it.severity == 'MEDIUM' }.size()
                    def highViolations = parsedJSON.results.violations.findAll { it.severity == 'HIGH' }.size()

                    // Fail pipeline for medium or high severity violations
                    if (mediumViolations > 0 || highViolations > 0) {
                        error("Terrascan found ${mediumViolations} medium and ${highViolations} high severity vulnerabilities. Check terrascan_output.json for details.")
                    } else {
                        echo "No critical vulnerabilities found. Terrascan passed successfully."
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
