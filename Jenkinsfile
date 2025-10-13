pipeline {
    agent any 
    stages {
        stage('Build') { 
            steps {
                sh 'dotnet restore' 
                sh 'dotnet build --no-restore' 
            }
        }
        stage('Test') { 
            steps {
                sh 'dotnet test --no-build --no-restore --collect "XPlat Code Coverage"'
            }
            post {
                always {
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: '**/*.xml']], sourceDirectories: [[path: 'MyWeatherAPI.Test/TestResults']])
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh 'dotnet publish MyWeatherAPI --no-restore -o published' 
            }
            post {
                success {
                    archiveArtifacts 'published/*.*'
                }
            }
        }
    }
    post {
        success {
            echo 'Build successful, now publishing artifacts...'
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'Ansible-Server', // Name configured in Jenkins global settings
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'published/**', // Path to the artifact in your workspace
                                remoteDirectory: '//opt//docker//MyWeatherAPI//published', // Target directory on the remote server
                                removePrefix: 'published/', // Optional: remove a prefix from source file paths
                                execCommand: '' // Optional: command to execute after transfer
                            )
                        ],
                        // Optional: execute commands before transfers
                        // execCommand: 'mkdir -p /path/on/remote/server'
                    )
                ]
            )
        }
        failure {
            echo 'Build failed, no publishing performed.'
        }
    }
}