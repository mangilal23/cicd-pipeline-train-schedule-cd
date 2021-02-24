pipeline {
agent any
    stages{
        stage('Build'){
            steps{
            echo "Running build automation"
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip', followSymlinks: false
            }
        }
        stage('Deploy Staging'){
            when {
            branch 'master'
            }
                steps{
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                sshPublisher 
                    failOnError: true, 
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging', 
                            sshCredentials: [
                                encryptedPassphrase: '$USERAPSS', 
                                username: '$USERNAME'
                            ], 
                            transfers: [
                                sshTransfer(
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule $$ rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule', 
                                    remoteDirectory: '/tmp', 
                                    remoteDirectorySDF: false, 
                                    removePrefix: 'dist/', 
                                    sourceFiles: 'dist/trainSchedule.zip'
                                )
                            ], 
                            usePromotionTimestamp: false, 
                            useWorkspaceInPromotion: false, 
                            verbose: false
                        )
                    ]
}
                }
            }
        }
    }
