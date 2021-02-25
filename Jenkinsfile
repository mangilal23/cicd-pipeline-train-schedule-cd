pipeline {
agent any
    stages{
        stage('Build'){
            steps{
            echo "Running build automation"
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Deploy Staging'){
            when {
            branch 'master'
            }
                steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                sshPublisher(
                    failOnError: true, 
                    continueOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging', 
                            sshCredentials: [
                                username: "$USERNAME",
                                encryptedPassphrase: "$USERPASS"
                                
                            ], 
                            transfers: [
                                sshTransfer(
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule', 
                                    remoteDirectory: '/tmp', 
                                    remoteDirectorySDF: false, 
                                    removePrefix: 'dist/', 
                                    sourceFiles: 'dist/trainSchedule.zip'
                                )
                            ]
                        )
                    ]
                    )
}
                }
            stage('Prod Deploy') {
                when {
                branch 'master'
                }
                input 'Have you tested application in staging environment ?'
                steps{
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
    // some block
                  sshPublisher(
                      publishers: [
                          sshPublisherDesc(
                              configName: 'prod',
                              transfers: [
                                  sshTransfer(
                                    
                                      execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule',
                                      remoteDirectory: '/tmp', 
                                      removePrefix: 'dist/', 
                                      sourceFiles: 'dist/trainSchedule.zip'
                                  )
                              ]
                          )])  
}
                }
            }
            
            }
        }
    }
