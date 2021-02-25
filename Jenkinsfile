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
                                    remoteDirectory: '/tmp', 
                                    remoteDirectorySDF: false, 
                                    removePrefix: 'dist/', 
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                )
                            ]
                        )
                    ]
                    )
                }
            }
        }
            stage('Prod Deploy') {
                when {
                branch 'master'
                }
                steps{
                    input 'Does the staging environment look OK?'
                    milestone(1)
                    withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'USERPASS', usernameVariable: 'USERNAME')]) {
                    sshPublisher(
                      publishers: [
                          sshPublisherDesc(
                              configName: 'prod',
                              sshCredentials: [
                                username: "$USERNAME",
                                encryptedPassphrase: "$USERPASS"  
                            ], 
                              transfers: [
                                  sshTransfer(
                                      remoteDirectory: '/tmp', 
                                      removePrefix: 'dist/', 
                                      sourceFiles: 'dist/trainSchedule.zip',
                                      execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                  )
                              ]
                          )
                        ]
                    )  
                }
            }
        }  
    }
}
