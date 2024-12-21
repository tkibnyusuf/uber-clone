pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                // Clone your repository
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }
        stage('Run Terrascan') {
            steps {
                script {
                    // Run Terrascan and save the exit status
                    def scanStatus = sh(
                        script: '''
                        docker run --rm -v /var/lib/jenkins/workspace/checkov_terraform:/iac accurics/terrascan:latest scan -d /iac/EKS_Terraform > terrascan_output.json
                        ''',
                        returnStatus: true
                    )

                    // Archive Terrascan results for analysis
                    archiveArtifacts artifacts: 'terrascan_output.json', allowEmptyArchive: true

                    // Fail the pipeline if vulnerabilities are found
                    if (scanStatus != 0) {
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
