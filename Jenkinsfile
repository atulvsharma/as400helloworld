pipeline {
    agent any
    environment {
        IBM_HOST = '150.238.118.254'           // Replace with your public IP or DNS
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

        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh """
                        ssh DEVUSR@150.238.118.254 '
                        /QOpenSys/pkgs/bin/rpg-analyzer -src /home/DEVUSR/as400helloworld/qrpglesrc -report /home/DEVUSR/sonar-report.json
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

        stage('Upload Code to IBM i') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        ssh $IBM_USER@$IBM_HOST "rm -rf $REMOTE_DIR && mkdir -p $REMOTE_DIR"
                        scp -r * $IBM_USER@$IBM_HOST:$REMOTE_DIR
                    '''
                }
            }
        }

        stage('Build on IBM i') {
            steps {
                sshagent(credentials: ['ibmi-ssh-creds-id']) {
                    sh '''
                        ssh $IBM_USER@$IBM_HOST "cd $REMOTE_DIR && /QOpenSys/pkgs/bin/gmake BIN_LIB=CMPSYS OPT=*EVENTF"
                    '''
                }
            }
        }

    }
}
