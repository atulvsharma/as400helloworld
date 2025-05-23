pipeline {
    agent any
    environment {
        IBM_HOST = '150.238.118.254'           // Public IP or DNS
        IBM_USER = 'DEVUSR'                    // Your IBM i username
        REMOTE_DIR = "/home/${IBM_USER}/as400helloworld"
    }
    stages {
         stage('Prepare Host Key') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        mkdir -p ~/.ssh
                        ssh-keyscan -H $IBM_HOST >> ~/.ssh/known_hosts
                    '''
                }
            }
        }

/*        stage('Static Code Analysis') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    withSonarQubeEnv('sonar') {
                        sh """
                            ssh DEVUSR@150.238.118.254 '
                            /QOpenSys/pkgs/bin/rpg-analyzer -src DEVUSR/qrpglesrc -report /home/DEVUSR/sonar-report.json
                            '

                            scp DEVUSR@150.238.118.254:/home/DEVUSR/sonar-report.json sonar-report.json

                            sonar-scanner \
                            -Dsonar.projectKey=AS400_HelloWorld \
                            -Dsonar.sources=. \
                            -Dsonar.externalIssuesReportPaths=sonar-report.json
                            """
                        }
                    }
                }
            } */

        stage('Upload Code to Dev-machine') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        ssh $IBM_USER@$IBM_HOST "rm -rf $REMOTE_DIR && mkdir -p $REMOTE_DIR"
                        scp -r * $IBM_USER@$IBM_HOST:$REMOTE_DIR
                    '''
                }
            }
        }

        stage('Build the artifact') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        ssh $IBM_USER@$IBM_HOST "cd $REMOTE_DIR && /QOpenSys/pkgs/bin/gmake BIN_LIB=CMPSYS OPT=*EVENTF"
                    '''
                }
            }
        }

        stage('Deploy to Dev machine') {
            steps {
                    sshagent(credentials: ['ibmi-ssh-creds-id']) {
                        sh '''
                            # On IBM i: copy/move into production library
                            ssh $IBM_USER@$IBM_HOST "system 'DLTPGM PGM(PRODLIB/HELLOWORLD)'; system 'CRTDUPOBJ OBJ(HELLOWORLD) FROMLIB(CMPSYS) OBJTYPE(*PGM) TOLIB(PRODLIB) NEWOBJ(HELLOWORLD)'"
                            '''
                        }
                    }
        }


        stage('List Files') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        ssh $IBM_USER@$IBM_HOST "ls -l /home/$IBM_USER/as400helloworld"
                        '''
                        }
                }
            }

        stage('Fetch Build Artifacts from Dev machine') {
            steps {
                    sshagent(credentials: ['ibmi-ssh-creds-id']) {
                        sh '''
                            mkdir -p ibmi-artifacts
                            scp $IBM_USER@$IBM_HOST:/home/$IBM_USER/as400helloworld/**/*.pgm ibmi-artifacts/ || true
                            scp $IBM_USER@$IBM_HOST:/home/$IBM_USER/as400helloworld/**/*.log ibmi-artifacts/ || true
                            scp $IBM_USER@$IBM_HOST:/home/$IBM_USER/as400helloworld/**/*.lst ibmi-artifacts/ || true
                            '''
                        }
                    }
            }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'ibmi-artifacts/**/*.*', allowEmptyArchive: true
                  }
            }
    }
}
