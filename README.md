
# **Capstone project – Orbit Bank**

**Objective**: To deploy a banking application on a Kubernetes cluster from Docker Hub.

**Tools to use:**

1. Jenkins
2. Github
3. Docker Hub
4. Ansible
5. Kubernetes

**Description**

Orbit Bank is one of the leading banking and financial service providers and is facing challenges in managing their monolithic applications and experiencing downtime during deployment. The company needs to develop an online banking application that provides private banks with a global accounting foundation, offering electronic banking services to all private banks, and enable private bank clients to carry out their daily transactions.

To address these issues, the company has decided to transition to a microservices architecture and implement a DevOps pipeline workflow using Jenkins, Ansible playbook, and Kubernetes cluster to deploy container on Docker Hub.

**Task (Activities)** 

1. Create the Dockerfile, Jenkinsfile, Ansible playbook, and the source file of the static website and upload it on the GitHub repository 
2. Create Jenkins pipeline to perform continuous integration and deployment for a Docker container 
3. Set up Docker Hub 
4. Set up Kubernetes cluster and configure deployment stage in the pipeline 
5. Configure Ansible playbook to deploy container on Docker Host 
6. Execute Jenkins build 
7. Access deployed application on a Docker container


## **Steps performed:**

## **1. Initial testing**

Before we use jenkins for the continous integration and deployment, let us just test the build of java application of source code using maven and use the jar file to test the creation of docker image manually

I have saved the source code provided by simplilearn in my github repo :

<https://github.com/kotianrakshith/CapstoneProject1>

First we will add the required plugins in the jenkins

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.001.png)

Ohers we can install later incase needed.(Please note that I have added many plugins in the jenkins for the project as we progress but are not documented.)

In the tools add Maven:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.002.png)

Now we create a new test freestyle project

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.003.png)

Give a general discription:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.004.png)

In the repository give the link:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.005.png)

Provide build steps:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.006.png)

Then we can save and build run, and we can see it has run succesfully![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.007.png)

This means that build works correctly.

Now we can proceed with the other tests and configurations before we configure the full pipeline.

Jenkins file we will create in later step by testing each step one by one in pipeline

## **2. DockerHub Setup**

Regarding Dockerhub, I already have a dockerhub account with id: **kotianrakshith**

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.008.png)

So I will use the same account.

For the Dockerfile, I will use below code :

Dockerfile: <https://github.com/kotianrakshith/CapstoneProject1/blob/main/Dockerfile>

## **3.Setup kubernetes cluster:**

We have three nodes with us, we will chose the system with jenkins installed as master and other two nodes as node 1 and node 2. We will rename it as master, worker node1 and worker node2:

Master:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.009.png)

Node1:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.010.png)

Node2:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.011.png)

Now lets do docker configuration in all the three nodes:
```
cat <<EOF | sudo tee /etc/docker/daemon.json

{

"exec-opts": ["native.cgroupdriver=systemd"],

"log-driver": "json-file",

"log-opts": {

"max-size": "100m"

},

"storage-driver": "overlay2"

}

EOF
```

```
sudo systemctl enable docker

sudo systemctl daemon-reload

sudo systemctl restart docker

sudo swapoff -a

```
![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.012.png)

**(same output screen in all three nodes)**

Now we will do master node initilaization using command:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.013.png)

From here we will copy the link it provides for the worker node initialization:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.014.png)

Then we will proceed to configure master with changing config permissions:

```
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

cat ~/.kube/config
```

With the last cat step you should be able to can view config file:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.015.png)

Then we will install a CNI, here we are chosing calico 

`kubectl apply -f <https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml>`

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.016.png)

Now we can do the worker node initialization using the command we copied:

Worker node 1:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.017.png)

Worker node 2:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.018.png)

Now in the worker node if we run command `kubectl get nodes` we should see all three nodes:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.019.png)

## **4. Create Jenkins pipeline script stage by stage**

Now lets create the jenkins file by creating test pipline with each step one by one:

As our build is already sucessfull lets try build and creating docker image

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.020.png)

We will chose this as a github project and we will give our github repository link:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.021.png)

For the first stage we will checkout the git and use mvn clean install to build the jar file

To get the checkout script we are using pipeline syntax to populate the script

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.022.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.023.png)

Please note that I’m using terminology testapp in the script which I will later change for our actual build      

With this step we can add below script:

```
stage('Build Maven'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/kotianrakshith/CapstoneProject1']])
                sh 'mvn clean install'
            }
        }
```
For the docker image build we are using below script:
```
  stage('Build Docker Image'){
            steps{
                script{
                    sh 'docker build -t kotianrakshith/testapp .'
                }
            }
        }
```

For now we will test this in the pipeline to see if it works till here:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.024.png)

As we had  issues with docker build, I have run the below command and reboot the system to give jenkins permission to run docker command:

`sudo usermod -a -G docker jenkins`

This solves the issue and maven build and docker image build is completed successfuly

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.025.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.026.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.027.png)

Now I will add the next stage, that is uploading the docker image to the docker hub

As we need password to login I will be creating access token instead of the password.

Usig pipeline variable I’m binding the password to a variable

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.028.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.029.png)

With this we can write the script as below:

```
stage('Push Docker Image to Dockerhub'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'dockerhubpwd', variable: 'dockerhubpassword')]) {
                    sh 'docker login -u kotianrakshith -p ${dockerhubpassword}'

                    sh 'docker push kotianrakshith/testapp'
                    }
                }
            }
        }

```
![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.030.png)

The pipeline build was successfull till this stage:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.031.png)

In the dockerhub we can see the testapp image added:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.032.png)

Now the last build stage is executing the ansible playbook to deploy the container in kubernetes using the docker image.

For the authorization we will add the jenkins user in the file /etc/sudoers

`sudo vim /etc/sudoers`

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.033.png)

As our initial ansible script failed we are using pipline syntax to pass the kube config file in to the jenkins workspace so it can have acccess to run the kubernes commands.

We have modified ansible workbook as below:

<https://github.com/kotianrakshith/CapstoneProject1/blob/main/kubernetesDeploy.yaml>

We have added the kuberntes deployment as a file kubewebapp.yaml in github:

<https://github.com/kotianrakshith/CapstoneProject1/blob/main/kubewebapp.yaml>

Now we use pipeline syntax to get the syntax with kube config file:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.034.png)


With this I have written the below stage script below:
```
stage('Execute Ansible Playbook'){
            steps{
                withCredentials([kubeconfigContent(credentialsId: 'Kubernetes', variable: 'KUBECONFIG_CONTENT')]) {
                    sh '''echo "$KUBECONFIG_CONTENT" > kubeconfig '''
                    sh 'ansible-playbook kubernetesDeploy.yaml'
                    sh 'rm kubeconfig'
              
                }
            }
        }
```

We can see that all our test is complete 

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.035.png)

And that test app is deployed in kubernets

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.036.png)

## **5. Executing complete Jenkins pipelin Build**

Now let us combine all in a Jenkins file and give correct terminology in all the file:

We are naming the app as orbitbankapp.

We have corrected all the files with the correct name and uploaded in github. Below are the links:

Jekins file link: <https://github.com/kotianrakshith/CapstoneProject1/blob/main/Jenkinsfile>

Dockerfile link: <https://github.com/kotianrakshith/CapstoneProject1/blob/main/Dockerfile>

Kubernets file link: <https://github.com/kotianrakshith/CapstoneProject1/blob/main/kubewebapp.yaml>

Ansible playbook link: <https://github.com/kotianrakshith/CapstoneProject1/blob/main/kubernetesDeploy.yaml>

Total Github link: <https://github.com/kotianrakshith/CapstoneProject1>

Now we have created Jenkins file we can create our actual project pipeline using the file from Github:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.037.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.038.png)

In the pipeline we are choosing pipeline script from SCM:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.039.png)

Give the correct branch name and Jenkins file name and save

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.040.png)

Once saved you can build the pipeline:

As we have tested all the script before it should work as expected and build correctly.

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.041.png)

Now we can check dockerhub if the image has been uploaded:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.042.png)

We can see our orbitbank app image

Now we can check in kubernetes if the deployment and service are present:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.043.png)

As we can see out app in the kubernets is running and service is also exposed in the nodeport 32000

## **6. Checking the deployment**

As the instruction provided with the source code we will use the below url to check the app:

_http://localhost:<port>/bank-api/swagger-ui.html_

Here as our node port is 32000

We will use the below url to check 

<http://localhost:32000/bank-api/swagger-ui.html>

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.044.png)

As we can see application is loading correctly:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.045.png)

And we are able to navigate:

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.046.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.047.png)

![](/readmeimages/Aspose.Words.1a61bbc6-95dc-4183-8ed2-178bd4c269d3.048.png)

So that concludes the project, we can improve on this project by making this an automated build by using poll scm or by using webhooks so it will run whenever there is a build made. But you can also click on build now whenever there is a change done and it should deploy the updated application to the kubernetes.

