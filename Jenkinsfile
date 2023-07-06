pipeline{
    agent any
    tools{
        maven 'Maven'
    }
    stages{
        stage('Build Maven'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/kotianrakshith/CapstoneProject1']])
                sh 'mvn clean install'
            }
        }
        stage('Build Docker Image'){
            steps{
                script{
                    sh 'docker build -t kotianrakshith/orbitbankapp .'
                }
            }
        }
        stage('Push Docker Image to Dockerhub'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpassword')]) {
                    sh 'docker login -u kotianrakshith -p ${dockerhubpassword}'

                    sh 'docker push kotianrakshith/orbitbankapp'
                    }
                }
            }
        }
        stage('Execute Ansible Playbook'){
            steps{
                withCredentials([kubeconfigContent(credentialsId: 'Kubernetes', variable: 'KUBECONFIG_CONTENT')]) {
                    sh '''echo "$KUBECONFIG_CONTENT" > kubeconfig '''
                    sh 'ansible-playbook kubernetesDeploy.yaml'
                    sh 'rm kubeconfig'
              
                }
            }
        }
    }
}
