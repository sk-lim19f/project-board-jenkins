pipeline{
    agent any

    trigger{
        pollSCM('*/3* * * *')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsAccessKeyId')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.'
    }

    stages{
        //Repository Install
        stage('Prepare') {
            agent any
            steps {
                echo 'Pulling Repository'

                git url: 'https://github.com/sk-lim19f/temp.git',
                    branch: 'master'
                    credentialsId: 'jenkinsgit'
            }
        }

        post {
            //If Gradle was able to run the tests, even if some of the test
            //failed, record ther test results and archive the jar file.
            success {
                echo 'Successfully Cloned Repository!!'
            }

            always {
                echo "I tried..."
            }

            cleanup{
                echo "after all other post condition"
            }
        }
        stage('Gredle test'){
            sh"""
            cd project-build
            chmod +x gradlew
            ./gradlew clean test
            """
        }

        stage('Gredle Build'){
            sh"""
            cd project-build
            chmod +x gradlew
            ./gradlew clean build
            """
        }

        stage('Ansible Deploy'){
            steps{
                script{

                    MY_KEYPAIR_NAME = ""
                    MY_APP_PRIVATE_IP = "" 

                    sh"""
                        git clone https://github.com/sk-lim19f/project-board.git
                        cd  project-build

                        sed -i 's/MY_KEYPAIR_NAME/${MY_KEYPAIR_NAME}/g' hosts/hosts
                        sed -i 's/MY_APP_PRIVATE_IP/${MY_APP_PRIVATE_IP}/g' hosts/hosts

                        ansible-playbook deploy_backend.yml -i ./hosts/hosts --extra-vars "deploy_hosts=app" 
                    """

                }
            }
        }
    }
}