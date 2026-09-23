pipeline {
    agent any

    environment {
        ARTIFACT_DIR = '/opt/jenkins-artifacts'
        NEXUS_URL = 'http://192.168.190.136:8081'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Failure') {
            steps {
                sh '''
                    echo "Testing Jenkins failure notification"
                    exit 1
                '''
            }
        }

        stage('Build Info') {
            steps {
                sh '''
                    echo "=== Jenkins Pipeline Started ==="
                    echo "Job: $JOB_NAME"
                    echo "Build Number: $BUILD_NUMBER"
                    echo "Workspace: $WORKSPACE"

                    hostname
                    whoami
                    date
                '''
            }
        }
        
        stage('Create Artifact') {
            steps {
                sh '''
                    set -eu

                    rm -rf output
                    mkdir -p output

                    TIMESTAMP=$(date +%Y%m%d-%H%M%S)
                    ARTIFACT_NAME="app-${BUILD_NUMBER}-${TIMESTAMP}.txt"

                    echo "Pipeline as Code - Version 2" > "output/$ARTIFACT_NAME"
                    echo "Build Number: $BUILD_NUMBER" >> "output/$ARTIFACT_NAME"
                    echo "Build Time: $TIMESTAMP" >> "output/$ARTIFACT_NAME"

                    echo "$ARTIFACT_NAME" > artifact-name.txt

                    echo "Created artifact:"
                    ls -l output/
                '''
            }
        }

        stage('Copy Artifact') {
            steps {
                sh '''
                    set -eu

                    ARTIFACT_NAME=$(cat artifact-name.txt)

                    cp "output/$ARTIFACT_NAME" "$ARTIFACT_DIR/"

                    echo "Artifact copied successfully."
                    ls -l "$ARTIFACT_DIR/$ARTIFACT_NAME"
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'output/*.txt', fingerprint: true
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-credentials',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -eu
                        set +x

                        ARTIFACT_NAME=$(cat artifact-name.txt)

                        echo "Uploading artifact to Nexus..."

                        curl --fail --silent --show-error \
                            --user "$NEXUS_USER:$NEXUS_PASSWORD" \
                            --upload-file "output/$ARTIFACT_NAME" \
                            "$NEXUS_URL/repository/jenkins-artifacts/$ARTIFACT_NAME"

                        echo "Artifact uploaded successfully to Nexus."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
post {
    success {
        echo 'Pipeline completed successfully.'
        emailext(
            to: 'swethahp123@outlook.com',
            subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                Jenkins build completed successfully.

                Job: ${env.JOB_NAME}
                Build: ${env.BUILD_NUMBER}
                Build URL: ${env.BUILD_URL}
            """
        )
    }

    failure {
        echo 'Pipeline failed. Check the console output.'
        emailext(
            to: 'swethahp123@outlook.com',
            subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
                Jenkins build failed.

                Job: ${env.JOB_NAME}
                Build: ${env.BUILD_NUMBER}
                Build URL: ${env.BUILD_URL}

                Check the Jenkins console output for details.
            """
        )
    }

    always {
        echo 'Pipeline execution finished.'
    }
}