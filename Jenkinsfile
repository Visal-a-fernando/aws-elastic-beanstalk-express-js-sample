pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
                echo 'Install Dependencies Finished'

                sh 'npm install snyk --save-dev'
                echo 'Snyk Installed'

                withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Finished'
                }
            }
        }

        stage('Security Scan') {
            steps {
                 script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json --ignore-policy', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Synk Vulnerabilities found! Check snyk-report.json.")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }

                echo 'Snyk Scan Completed'
            }
            post {
                success {
                    echo 'Sync Security Scan passed!'
                }
                failure {
                    echo 'Synk Failed.'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh 'npm install supertest'
                    sh 'npm test'
                    echo 'Test completed.'
                }
            }
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Failed. Check logs for details.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Completed.'
        }
    }
}
