
pipeline {
    agent any

    environment {
        ANSIBLE_SERVER_IP = '46.101.229.169'
        ANSIBLE_USER = 'root'
        SSH_PRIVATE_KEY_PATH = 'ansible-jenkins-key-pair.pem'
    }

    stages {
        stage('copy files to ansible server') {
            steps {
                script {
                    echo 'Copying all necessary files to ansible server...'

                    sshagent(['ansible-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no -r ansible/* ${ANSIBLE_USER}@${ANSIBLE_SERVER_IP}:/${ANSIBLE_USER}/"

                        withCredentials([sshUserPrivateKey(credentialsId: 'server-ssh-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                            sh 'scp -o StrictHostKeyChecking=no "$keyfile" "${ANSIBLE_USER}@${ANSIBLE_SERVER_IP}:/${ANSIBLE_USER}/${SSH_PRIVATE_KEY_PATH}"'
                        }
                    }
                }
            }
        }

        stage("execute ansible playbook") {
            steps {
                script {
                    echo "calling ansible playbook to configure ec2 instances"

                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = ANSIBLE_SERVER_IP
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
                        remote.user = user
                        remote.identityFile = keyfile
                        sh '''ssh -o StrictHostKeyChecking=no -i "$keyfile" "$user@$ANSIBLE_SERVER_IP" "ls -l; pwd"'''
                        sh '''ssh -o StrictHostKeyChecking=no -i "$keyfile" "$user@$ANSIBLE_SERVER_IP" "ansible-playbook deploy-docker-new-user.yaml"'''
                    }
                }
            }
        }
    }
}
