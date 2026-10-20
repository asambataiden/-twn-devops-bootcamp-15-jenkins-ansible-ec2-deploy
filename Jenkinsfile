
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
                    echo "Calling Ansible playbook to configure EC2 instances"

                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = ANSIBLE_SERVER_IP
                    remote.allowAnyHosts = true

                    withCredentials([
                        sshUserPrivateKey(
                            credentialsId: 'ansible-server-key',
                            keyFileVariable: 'keyfile',
                            usernameVariable: 'user'
                        )
                    ]) {
                        remote.user = user
                        remote.identityFile = keyfile

                        sshCommand remote: remote, command: "ls -l; pwd"

                        withCredentials([
                            string(
                                credentialsId: 'ansible-vault-password',
                                variable: 'VAULT_PASSWORD'
                            )
                        ]) {
                            sh '''
                                set +x
                                set -eu

                                printf '%s\\n' "$VAULT_PASSWORD" | \
                                ssh -T \
                                  -o BatchMode=yes \
                                  -o StrictHostKeyChecking=yes \
                                  -i "$keyfile" \
                                  "$user@$ANSIBLE_SERVER_IP" \
                                  'cd /root && ansible-playbook --vault-password-file /dev/stdin deploy-ansible-roles-docker-new-user.yaml'
                            '''
                        }
                    }
                }
            }
        }
    }
}
