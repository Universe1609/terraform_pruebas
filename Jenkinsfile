pipeline {
    agent any

    stages {
        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage("Checkout Git") {
            steps {
                git branch: 'master', url: 'https://github.com/Universe1609/terraform_pruebas'
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('./Terraform') {
                    sh 'terraform init' //backend s3 initialized
                    
                    sh 'terraform plan'
            }
        }
        }

        stage('Apply Terraform') {
            steps {
                 dir('Terraform') {
                    script {
                        try {
                            sh 'terraform apply -auto-approve'
                        } catch (Exception e) {
                            // If terraform apply fails, run terraform destroy to clean up
                            echo "Error durante Terraform Apply: ${e.getMessage()}"
                            echo "Attempting to destroy any resources that were created..."
                            sh 'terraform destroy -auto-approve'
                            // Rethrow the exception to mark the build as failed
                            throw e
                        }
                    }
                }
            }
        }

        stage('Generate Ansible Inventory') {
            steps {
                script {
                    def ipAddress = sh(script: "terraform output -raw ec2_instance_ip", returnStdout: true).trim()
                    echo "Terraform output for ec2_instance_ip: ${ipAddress}"
                    if (ipAddress == null || ipAddress.isEmpty()) {
                        echo "IP no encontrada."
                    } else {
                        def inventoryContent = "[ec2_instance]\n${ipAddress} ansible_user=ubuntu ansible_ssh_private_key_file=\${SSH_KEY}"
                        writeFile file: 'Ansible/inventory', text: inventoryContent
                        sh 'cat Ansible/inventory'
                        }
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([file(credentialsId: 'SSH_KEY', variable: 'SSH_KEY')]) {
                    sh 'ansible-playbook -i Ansible/inventory Ansible/playbooks/ec2_jenkins.yml --private-key $SSH_KEY -u ubuntu'
                }
            }
        }
    }
}
