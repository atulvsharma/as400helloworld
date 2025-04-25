pipeline {
    agent any
    environment {
        IBM_HOST = '150.238.118.254'           // Replace with your public IP or DNS
        IBM_USER = 'DEVUSR'                    // Your IBM i username
        REMOTE_DIR = "/home/${IBM_USER}/as400helloworld"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: 'atulvsharma@github.com:atulvsharma/as400helloworld.git'
            }
        }

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
