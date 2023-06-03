
# Kubernetes DevOps Workflow with GitLab, Jenkins, and ASP.NET Core

This project sets up a Minikube Kubernetes cluster on your machine,   
and a Jenkins pipeline to build and deploy the web app.  
It includes Helm Charts for GitLab, Jenkins, and an ASP.NET Core web application.


## Table of Contents

- [Prerequisites](#prerequisites)

- [Generic Setup](#generic-setup)

- [Set up a Minikube k8s cluster](#set-up-a-minikube-k8s-cluster)

- [Set up storage for GitLab and Jenkins](#set-up-storage-for-gitlab-and-jenkins)

- [Set up the GitLab server](#set-up-the-gitlab-server)

- [Set up the Jenkins server](#set-up-the-jenkins-server)

- [Set up the Jenkins build agent](#set-up-the-jenkins-build-agent)

- [Configuration for GitLab and Jenkins](#configuration-for-gitlab-and-jenkins)

- [Configuration for a ASP.NET Core web app deployment](#configuration-for-a-asp-net-core-web-app-deployment)

- [Notes](#notes)

- [Troubleshooting](#troubleshooting)


## Prerequisites

This was set up and ran on an Ubuntu 20.04 OS machine -
so the instructions will fit this OS.


Before you can set up and run this project, please ensure that you have the following installed:

\
DockerHub - have a DockerHub account and a repo to push and pull from.

\
Git - [link](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
```bash
sudo apt install git
```

cURL - [link](https://everything.curl.dev/get)
```bash
sudo apt install curl
```

OpenSSH - [link](https://www.cyberciti.biz/faq/ubuntu-linux-install-openssh-server/)
```bash
sudo apt install openssh-server
```

Docker - [link](https://docs.docker.com/engine/install/)
```bash
sudo apt install docker.io
sudo usermod -a -G docker ${USER}
newgrp docker
sudo systemctl enable docker.service
```

Helm - [link](https://helm.sh/docs/intro/install/)
```bash
sudo curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Kubectl - [link](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
```bash
sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Minikube - [link](https://minikube.sigs.k8s.io/docs/start/)
```bash
sudo curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```


## Generic Setup

- Create a directory  where you would like to clone this repo to:
```bash
cd /path/to/create/dir
mkdir <name>
```

- Clone the repo:
```bash
git clone https://github.com/itsrayto/Elta-home-assignment.git
```


## Set up a Minikube k8s cluster

- From a terminal with admin access, run:
```bash
minikube start
```

- You should see your minikube cluster running as a container, check it with:
```bash
docker ps
```

- Now you can use kubectl to access the cluster.
```bash
kubectl get pods -A
```     



- **Note**: to access the GitLab and Jenkins servers   
    you will use the Minikube ip and the corresponding node port.

- Run this to get the ip:
```bash
minikube ip
```

- We should also enable the NGINX Ingress controller for the web app.
    ```bash
    minikube addons enable ingress
    ```
    - Verify that the NGINX Ingress controller is running:
        ```bash
        kubectl get pods -n ingress-nginx
        ```


## Set up storage for GitLab and Jenkins

- We will create 2 folders inside the Minikube container that the servers will use as storage:   

```bash
docker exec -it minikube bash

# inside the Minikube container:
cd /

cd home/
mkdir gitlab
cd gitlab/
mkdir data

cd /

cd home/
mkdir jenkins
cd jenkins/
mkdir data
```


## Set up the GitLab server

- Go to the 'gitlab' folder from this repo and apply the 'gitlab' namespace:
```bash
kubectl apply -f namespace.yaml
```

- Then go to the 'gitlab-chart' folder and install the chart in the 'gitlab' ns:
```bash
helm install gitlab . -n gitlab --values values.yaml
```

- Access the GitLab server through http://192.168.49.2:30100/ (minikube-ip:node-port)
- The username is: root  
    The password is: git1test23 (as defined in the chart)  
   


- Create a new project

    ![alt text](https://i.imgur.com/kWEemDL.png)

    ![alt text](https://i.imgur.com/mgwC1tN.png)

- Choose a project name and visibility level (private/public).

- GitLab should be ready now!


## Set up the Jenkins server

- Go to the 'jenkins' folder from this repo and apply the 'devops' namespace
```bash
kubectl apply -f namespace.yaml
```

- Then go to the 'jenkins-chart' folder and install the chart in the 'devops' ns
```bash
helm install jenkins . -n devops --values values.yaml
```

- Access the Jenkins server through http://192.168.49.2:30200/ (minikube-ip:node-port)

- You will be asked to Unlock Jenkins with the initialAdminPassword.  
    to get it run the following:
```bash
kubectl get pods -n devops
kubectl exec -it <jenkins-pod> -n devops -- cat /var/jenkins_home/secrets/initialAdminPassword
```
 
- We will install the suggested plugins and add what we need later.

    ![alt text](https://i.imgur.com/cDQl7iB.png)

- Create your Admin user

    ![alt text](https://i.imgur.com/o4rSufc.png)

- In the Instance Configuration leave it as the default

    ![alt text](https://i.imgur.com/a45pRDi.png)

- Jenkins should be ready now!


## Set up the Jenkins build agent

- Now we're going to set up Kuberenetes-based jenkins agent.

- First, we have to install the Kuberenetes plugin on Jenkins:

    - On the Jenkins server, select Manage Jenkins > Manage Plugins.

    - Install the Jenkins Kuberentes Plugin.

    ![alt text](https://i.imgur.com/UBBd35z.png)

    ![alt test](https://i.imgur.com/AlppaNC.png)

    ![alt text](https://i.imgur.com/WTRtvZB.png)

- Next, we create a Kuberenetes Cloud Configuration:
    - Once the plugin is installed,  
        go to Manage Jenkins > System Configuration > Nodes and Clouds.

    - On the left panel > Clouds.

    - Add a new Cloud > select Kuberenetes.

    - Confgiure the Cloud:  

        Since we have Jenkins inside the Kubernetes cluster with a service account to deploy the agent pods, we don’t have to mention the Kubernetes URL or certificate key.\
        However, to validate the connection using the service account, use the Test connection button as shown below. It should show a connected message if the Jenkins pod can connect to the Kubernetes master API.

    ![alt text](https://i.imgur.com/3STTYAA.png)


    - For Jenkins master running inside the cluster, you can use the Service endpoint of the Kubernetes cluster as the Jenkins URL because agents pods can connect to the cluster via internal service DNS.

        - The URL is derived using the following syntax:
            ```bash
            http://<service-name>.<namespace>.svc.cluster.local:8080
            ```
        - Also, add the POD label that can be used for grouping the containers.

            ![alt text](https://i.imgur.com/Iq1kmVl.png)


- You can have a POD template in the Cloud configurations, but when it comes to actual project pipelines, it is better to have the POD templates in the Jenkinsfile

    - Here is what you should know about the POD template:

        - By default, a JNLP container image is used by the plugin to connect to the Jenkins server. You can override with a custom JNLP image provided you give the name jnlp in the container template.

        - You can have multiple container templates in a single pod template. Then, each container can be used in different pipeline stages.

        - POD_LABEL will assign a random build label to the pod when the build is triggered. You cannot give any other names other than POD_LABEL.

        - **Note**: You can declare a podTemplate in a yaml format, that's what I'm doing in my Jenkinsfile.


## Configuration for GitLab and Jenkins


#### Grant Jenkins access to the GitLab project

- Create a personal access token:
    - In the upper-right corner, select your avatar.

    - Select Edit profile.

    - On the left sidebar, select Access Tokens.

    - Enter a name and expiry date for the token.

    - If you do not enter an expiry date, the expiry date is automatically set to 365 days later than the current date.

    - Select the desired scopes.

    - Select Create personal access token.

    - Save the personal access token somewhere safe. After you leave the page, you no longer have access to the token.

- Set the access token scope to API.

- Copy the access token value to Configure the Jenkins server.

---
#### Configure the Jenkins server

- Install and configure the Jenkins plugin.  
    The plugin must be installed and configured to authorize the connection to GitLab.

- On the Jenkins server, select Manage Jenkins > Manage Plugins.

- Install the Jenkins GitLab Plugin.

    ![alt text](https://i.imgur.com/Yqu0qM6.png)

    ![alt test](https://i.imgur.com/AlppaNC.png)

    ![alt text](https://i.imgur.com/WTRtvZB.png)

- Select Manage Jenkins > System Configuration > System.

- In the GitLab section, select Enable authentication for ‘/project’ end-point.

- Select Add, then choose Jenkins Credential Provider.

- Select GitLab API token as the token type.

- In API Token, paste the value you copied from GitLab and select Add.

- Enter the GitLab server’s URL in GitLab host URL.

- To test the connection, select Test Connection.

    ![alt text](https://i.imgur.com/K705nfr.png)

    ![alt text](https://i.imgur.com/jM9dEiw.png)

---
#### Upload the needed files from this repo to our GitLab project

- Clone the GitLab project:
```bash
git clone http://192.168.49.2:30100/gitlab-instance-66f47545/web-app.git
```

- Copy the contents of 'gitlab-repo' from this repo to the folder of that clone.

- Add, commit and push the new files to the GitLab project:
    ```bash
    git add .
    git commit -m "gitlab-repo files added"
    git push
    ```

    - The GitLab credentials are the ones from the GitLab server set up.

---
#### Configure the Jenkins project

- Set up the Jenkins project you intend to run your build on.

- On your Jenkins instance, go to New Item.

- Enter the project’s name.

- Select Pipeline and select OK.

- Choose your GitLab connection from the dropdown list.

- Select Build when a change is pushed to GitLab.

- Select the following checkboxes:

    - Push Events

    - Closed Merge Request Events

- Specify how the build status is reported to GitLab:
    - If you created a Pipeline project, you must use a Jenkins Pipeline script to update the status on GitLab.

- In the Pipeline section:
    - Definition > Pipeline script from scm.
    - SCM > Git.
    - Repository URL > The URL from Clone with the right port.
    - Credentials > Create credentials of a username and a password:\
        useername: root\
        password: the personal access token from before

    ![alt text](https://i.imgur.com/yFHW9gB.png)

    ![alt text](https://i.imgur.com/OQwNOXt.png)

    ![alt text](https://i.imgur.com/dlqLKev.png)


---
#### Configure the GitLab project

- Configure a Jenkins integration

- On the top bar, select Main menu > Projects and find your project.

- On the left sidebar, select Settings > Integrations.

- Select Jenkins.

- Select the Active checkbox.

- Select the events you want GitLab to trigger a Jenkins build for:
    - Push
    - Merge request

- Enter the Jenkins server URL.

- Clear the Enable SSL verification checkbox to disable SSL verification.

- Enter the Project name.

- The project name should be URL-friendly, where spaces are replaced with underscores. To ensure the project name is valid, copy it from your browser’s address bar while viewing the Jenkins project.

- If your Jenkins server requires authentication, enter the Username and Password.

- Optional. Select Test settings.
    - **Note**: You will get 'Validations failed' in this step, here's why:  
        The reason is that the Jenkins URL points to the same IP address which GitLab is using(Minikube setup).  
        Such webhooks are forbidden by default and can be enabled in the GitLab settings Admin → Settings → Network → Outbound Requests → Allow requests to the local network from hooks and services

    ![alt text](https://i.imgur.com/Ksv99nB.png)

    - After you check this box  
        you will get 'Connection successful' when trying to Test Settings.

- Select Save changes.


## Configuration for a ASP NET Core web app deployment

- First, let's create a 'production' namespace for our web app to run on:
    - Go to the 'webapp-prod' folder from this repo and apply the 'production' namespace
        ```bash
            kubectl apply -f namespace.yaml
        ```

- We should also change to the correct 'user'@'ip' and <path/to/webapp-chart> in our Jenkinsfile:
    - To find your ip run:
        ```bash
        ifconfig
        ```

- After that, let's connect to our DockerHub account and set up an Access Token to push/pull the image during our Jenkins pipeline.

    - After logging in and creating a private repo:
        - Go to top right > click username > Account Settings  .
        - Go to Security > New Access Token.
        - Fill the details:

        ![alt text](https://i.imgur.com/d8CWv4k.png)

        - Copy the Access Token and save it
        

- Then go to your machine, and create a secret in 2 namespaces:  
    - The devops namespace - to let the Jenkins agent push into the DockerHub private repo with Kaniko.

    ```bash
    kubectl create secret docker-registry docker-credentials --docker-username=<your-docker-username> --docker-password=<your-docker-access-token> --namespace devops
    ```

    - The production namespace - so the web app production pod can pull the image from DockerHub.

    ```bash
    kubectl create secret docker-registry docker-credentials --docker-username=<your-docker-username> --docker-password=<your-docker-access-token> --namespace production
    ```

- Now let's take care of the deployment, we will need to set up an SSH Agent plugin on Jenkins.
    - On the Jenkins server, select Manage Jenkins > Manage Plugins.

    - Install the Jenkins SSH Agent Plugin.
    
    ![alt text](https://i.imgur.com/bSUqyNY.png)

    ![alt test](https://i.imgur.com/AlppaNC.png)

    ![alt text](https://i.imgur.com/WTRtvZB.png)

    - On your machine, run the following command to generate an SSH key:
    ```bash
    ssh-keygen -t rsa 
    ```
    - Now you got a private and a public key
        - The public key - by default is located in '~/.ssh/id_rsa.pub'.  
            you should put it in your machine's 'authorized_keys' file (found in  ~/.ssh/authorized_keys. create a new one if it doesn't exist).

        - The private key - by default is located in '~/.ssh/id_rsa'.  
            We're going to use it to create credenetials in Jenkins so the SSH Agent plugin can use them   
            when connecting from the agent pod to our machine through SSH during the Deploy stage:

            - Go to the Jenkins server > Manage Jenkins.
            
            - Go to Security > Credentials

            - Domain > (global)

            - On the right click on Add Credenetials

            ![alt text](https://i.imgur.com/uEuqMY6.png)

            - Copy the contects of the private key file to the Key field here

            ![alt text](https://i.imgur.com/cl0I9mb.png)
            
            - And done!


- Lastly, we'll make the Ingress work for our web app:
    - To be able to access the web app through 'the-legend.info' as we defined in the Ingress object,  
        we will need to add the following line to our '/etc/hosts' file:   

        ```
        192.168.49.2    the-legend.info
        ```  
        **Note**: The ip is the ip of Minikube


- That's it, now when we push to the GitLab project or build the project on Jenkins,   
        the webapp should be deployed through the Helm Chart and we'll be able to access it through:   

    **_the-legend.info_**


## Notes

#### Kaniko as a Docker image builder inside Kuberenetes

- Building Docker in Docker
        In CI, one of the main stages is to build the Docker images. In containerized builds, you can use Docker in the Docker workflow.

- But this approach has the following disadvantages.

    - The Docker build containers run in privileged mode. It is a big security concern and it is kind of an open door to malicious attacks.

    - Kubernetes removed Docker from its core. So, mounting docker.sock to host will not work in the future, unless you add a docker to all the Kubernetes Nodes.

- These issues can be resolved using Kaniko.


#### Build  Docker Image In Kubernetes Using Kaniko
- Kaniko is an open-source container image-building tool created by Google.

- It does not require privileged access to the host for building container images.

- Here is how Kaniko works:

    - There is a dedicated Kaniko executer image that builds the container images.  
        It is recommended to use the 'gcr.io/kaniko-project/executor' image to avoid any possible issues.  
        Because this image contains only static go binary and logic to push/pull images from/to registry.

    - Kaniko accepts three arguments: A Dockerfile, build context, and a remote Docker registry.

    - When you deploy the kaniko image, it reads the Dockerfile and extracts the base image file system using the FROM instruction.

    - Then, it executes each instruction from the Dockerfile and takes a snapshot in the userspace.

    - After each snapshot, kaniko appends only the changed image layers to the base image and updates the image metadata.  
        It happens for all the instructions in the Dockerfile.

    - Finally, it pushes the image to the given registry.
As you can see, all the image-building operations happen inside the Kaniko container’s userspace and it does not require any privileged access to the host.

## Troubleshooting

- Agent doesnt work - https://github.com/jenkinsci/docker-inbound-agent/issues/146

