pipeline {
    agent { label 'slave-name' }
    stages {
        stage('Running tests') {
            steps {
                cleanWs()

                script {
                    def myEnv;
                    def myEnvID;
                }

                sshagent(['deployer']) {
                    dir("roles/opsschol-ansible") {
                        checkout scm
                    }

                    script {
                        sh 'cp -r roles/opsschol-ansible/tests/* .'
                    }
                }

                script {
                    myEnv = docker.build('centos_sshd')
                    
                    myEnvID = myEnv.run('--privileged -d -p 0.0.0.0:2200:22', '/usr/sbin/sshd -D')
                    
                    echo 'Docker id:' + myEnvID;
                }

                sshagent(['deployer']) {
                    
                        ansiColor('xterm') {
                            sh '''
                                #!/bin/bash
                                export ANSIBLE_HOST_KEY_CHECKING=False
                                export ANSIBLE_FORCE_COLOR=true
                                ansible-playbook test.yml -i inventory  -e "ansible_port=2200" --diff
        
                            '''
                        }
                }

            }
        }
    }
    post {
        always {
            script {
                myEnvID.stop()
            }
        }
    }
}
