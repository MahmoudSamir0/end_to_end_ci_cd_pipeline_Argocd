# end_to_end_ci_cd_pipeline_Argocd
 ![](/screenshots/argocd.png)

## Project Overview
### Objectives:
The primary objectives of this project are to establish a robust CI/CD pipeline that automates the software delivery process and embraces modern best practices for containerized applications. 

### Workflow:
1. Developer Workflow:

    Developers work on application code locally and push changes to a designated private Git repository.

2. Continuous Integration (CI) Workflow:

    Jenkins, configured with webhooks or periodic polling, detects changes in the private Git repository.
    Upon detecting changes, Jenkins triggers a CI build, fetching the latest code, and building a Docker image.

3. Continuous Deployment (CD) Workflow:

    Successfully built Docker images are tagged and pushed to Amazon ECR by Jenkins.
    ArgoCD continuously monitors the private Git repository for changes in Kubernetes manifests.
    Upon detecting changes, ArgoCD automatically syncs the manifests with the desired state, deploying the updated application to the Amazon EKS cluster.

4. GitOps Principles:

    Declarative Configuration: Kubernetes manifests serve as the declarative configuration of the desired application state.

    Rollback Capabilities: ArgoCD provides easy rollback capabilities, enabling a quick return to a previous application state in case of issues.

### Infrastructure :

1. **Amazon EKS Cluster**: The applications are deployed to an Amazon EKS cluster, managed and orchestrated by Kubernetes. EKS provides a scalable and secure platform for running containerized applications.

### Monitoring :

1. **ArgoCD Dashboard**: ArgoCD offers a user-friendly dashboard for monitoring the deployed applications, providing visibility into the current and desired states.

1. implement monitoring for your automated pipeline. Use ArgoCD's built-in dashboards and logs to troubleshoot any issues that may arise during the deployment process.

## Getting Started

### Setup tools
#### install helm 
you will use Helm to install Argocd  with a stable _chart_.  Helm
is a package manager that makes it easy to configure and deploy Kubernetes
applications.

you have two way to install helm.

##### 1) From the Binary Releases 
1. Download and install the helm binary

    ```shell
    wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
    ```

1. Unzip the file to your local system:

    ```shell
    tar zxfv helm-v3.2.1-linux-amd64.tar.gz
    cp linux-amd64/helm .
    ```

1. Add the official stable repository.

    ```shell
    ./helm repo add stable https://kubernetes-charts.storage.googleapis.com
    ```

1. Ensure Helm is properly installed by running the following command. You
   should see version `v3.2.1` appear:

    ```shell
    ./helm version
    ```

    Output (do not copy):

    ```output
    version.BuildInfo{Version:"v3.2.1", GitCommit:"fe51cd1e31e6a202cba7dead9552a6d418ded79a", GitTreeState:"clean", GoVersion:"go1.13.10"}
    ```

##### 2) From Script
You can fetch that script, and then execute it locally. It's well documented so that you can read through it and understand what it is doing before you run it.

```shell
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

    chmod 700 get_helm.sh
    ./get_helm.sh   
```


#### Configure and Install argocd

```shell
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.8.4/manifests/install.yaml
```
1. Now, check that the argocd was created properly:

    ```shell
    kubectl get all -n argocd
    ```


    Output (do not copy):
    

    ```output
    NAME                                                    READY   STATUS    RESTARTS         AGE
    pod/argocd-application-controller-0                     1/1     Running   76 (4m30s ago)   52d
    pod/argocd-applicationset-controller-745cd84657-7krjt   1/1     Running   77 (4m30s ago)   52d
    pod/argocd-dex-server-684c58b4b5-nt7dq                  1/1     Running   76 (4m30s ago)   52d
    pod/argocd-notifications-controller-f5877f4fb-h6vdj     1/1     Running   130 (92s ago)    52d
    pod/argocd-redis-685866888c-m67bq                       1/1     Running   76 (4m30s ago)   52d
    pod/argocd-repo-server-76bc8c68b9-hp5g9                 1/1     Running   76 (4m30s ago)   52d
    pod/argocd-server-b456cd7d5-m5sgj                       1/1     Running   80 (4m30s ago)   52d

    NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    service/argocd-applicationset-controller          ClusterIP   10.108.241.94    <none>        7000/TCP,8080/TCP            52d
    service/argocd-dex-server                         ClusterIP   10.109.9.44      <none>        5556/TCP,5557/TCP,5558/TCP   52d
    service/argocd-metrics                            ClusterIP   10.98.79.66      <none>        8082/TCP                     52d
    service/argocd-notifications-controller-metrics   ClusterIP   10.98.77.84      <none>        9001/TCP                     52d
    service/argocd-redis                              ClusterIP   10.109.217.227   <none>        6379/TCP                     52d
    service/argocd-repo-server                        ClusterIP   10.111.152.33    <none>        8081/TCP,8084/TCP            52d
    service/argocd-server                             NodePort    10.102.41.161    <none>        80:31086/TCP,443:31549/TCP   52d
    service/argocd-server-metrics                     ClusterIP   10.104.119.26    <none>        8083/TCP                     52d

    ```
The Argo CD can be installed using [Helm](https://helm.sh/). The Helm chart is currently community maintained and available at
[argo-helm-repo](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd).

#### Connect to Argocd
1. The Jenkins chart will automatically create an admin password for you. To
   retrieve it, run:

    ```shell
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

2. To get to the Argocd user interface
   ```shell
    kubectl get svc -n argocd
    ```
    ```output
        NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    argocd-applicationset-controller          ClusterIP   10.108.241.94    <none>        7000/TCP,8080/TCP            53d
    argocd-dex-server                         ClusterIP   10.109.9.44      <none>        5556/TCP,5557/TCP,5558/TCP   53d
    argocd-metrics                            ClusterIP   10.98.79.66      <none>        8082/TCP                     53d
    argocd-notifications-controller-metrics   ClusterIP   10.98.77.84      <none>        9001/TCP                     53d
    argocd-redis                              ClusterIP   10.109.217.227   <none>        6379/TCP                     53d
    argocd-repo-server                        ClusterIP   10.111.152.33    <none>        8081/TCP,8084/TCP            53d
    argocd-server                             NodePort    10.102.41.161    <none>        80:31086/TCP,443:31549/TCP   53d
    argocd-server-metrics                     ClusterIP   10.104.119.26    <none>        8083/TCP                     53d
    ```
    get  argocd-server  Nodeport example **31549** you can change to Nodeport if there is cluster ip for argocd-server 

    ```shell
    kubectl edit service/argocd-server -n argocd
    ```
    ```output
        # Please edit the object below. Lines beginning with a '#' will be ignored,
    # and an empty file will abort the edit. If an error occurs while saving this file will be
    # reopened with the relevant failures.
    #
    apiVersion: v1
    kind: Service
    metadata:
    annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app.kubernetes.io/component":"server","app.kubernetes.io/name":"argocd-server","app.kubernetes.io/part-of":"argocd"},"name":"argocd-server","namespace":"argocd"},"spec":{"ports":[{"name":"http","port":80,"protocol":"TCP","targetPort":8080},{"name":"https","port":443,"protocol":"TCP","targetPort":8080}],"selector":{"app.kubernetes.io/name":"argocd-server"}}}
    creationTimestamp: "2023-09-25T20:34:13Z"
    labels:
        app.kubernetes.io/component: server
        app.kubernetes.io/name: argocd-server
        app.kubernetes.io/part-of: argocd
    name: argocd-server
    namespace: argocd
    resourceVersion: "51705"
    uid: 216a100c-a403-4719-8cb2-a9d1f682ab8e
    spec:
    clusterIP: 10.102.41.161
    clusterIPs:
    - 10.102.41.161
    externalTrafficPolicy: Cluster
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: http
        nodePort: 31086
        port: 80
        protocol: TCP
        targetPort: 8080
    - name: https
        nodePort: 31549
        port: 443
        protocol: TCP
        targetPort: 8080
    selector:
        app.kubernetes.io/name: argocd-server
    sessionAffinity: None
    type: NodePort   <--------- change it to NodePort 
    status:
    loadBalancer: {}

    ```
    You should now be able to log in with username `admin` and your auto generated
    password.

    ![](/screenshots/argo_ui_login.png)
    ![](/screenshots/argo_ui_login-1.png)

#### Configure and Install Jenkins
1. you can use helm to install Jenkins

   1. using cli
        ```shell
        helm repo add myjenkins https://charts.jenkins.io
        helm repo update    
        ```
        ```shell
        helm install my-jenkins jenkins/jenkins --version 4.6.5 --set controller.serviceType=NodePort    
        ```
   1. using argocd ui 
 ![](/screenshots/argo_jenkins_1.png)
 ![](/screenshots/argo_jenkins_2.png)
 ![](/screenshots/argo_jenkins_3.png)
 ![](/screenshots/argo_jenkins_4.png)
 now you app is ready

#### Connect to Argocd

```shell
kubectl get all -n jenkins    
```
```output
NAME                                 READY   STATUS    RESTARTS         AGE
pod/jenkins-0                        2/2     Running   29 (3h18m ago)   10d

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/jenkins             NodePort    10.98.170.80     <none>        8080:31599/TCP   10d
service/jenkins-agent       ClusterIP   10.102.195.223   <none>        50000/TCP        10d
```

1. The Jenkins chart will automatically create an admin password for you. To
   retrieve it, run:

    ```shell
    printf $(kubectl get secret --namespace jenkins jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
    ```

You should now be able to log in with username `admin` and your auto generated
password.

![](/screenshots/argo_jenkins_5.png)
![](/screenshots/argo_jenkins_6.png)

#### Configure and Install jenkins agent 
Dockerfile for jenkins agent
```dockerfile
FROM ubuntu
USER root
# create user
RUN useradd -m jenkins
RUN mkdir -p /var/jenkins_home
RUN chown -R jenkins:jenkins /var/jenkins_home
WORKDIR /home/jenkins
RUN apt update && apt dist-upgrade -y
# Install required packages
RUN apt install -y \
    git \
    apt-transport-https \
    curl \
    software-properties-common \
    unzip \
    openssh-server openssh-client \
    vim \
    ca-certificates \
    gnupg \
    lsb-release
# install docker
RUN mkdir -m 0755 -p /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN apt-get update
RUN apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# install kubectl
RUN curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
RUN echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
RUN apt-get update
RUN apt-get install -y kubectl
# install helm3
RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \
    && chmod 700 get_helm.sh \
    && ./get_helm.sh
# Add the HashiCorp GPG key
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add -

# Add the HashiCorp official Debian repository
RUN add-apt-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

# Install Terraform
RUN apt-get update && \
    apt-get install -y terraform

# Install AWS CLI
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm awscliv2.zip
# install nodejs
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs
# install java
RUN apt install -y openjdk-17-jdk
# install maven
RUN apt-get install -y maven
# expose port
# Cleanup
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/*
EXPOSE 22
# running process
ENTRYPOINT ["tail"]
CMD ["-f", "/dev/null"]
```


agent yaml files
1. Deployment

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: jenkins-agent
    namespace: jenkins
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: jenkins-agent
    template:
        metadata:
        labels:
            app: jenkins-agent
        spec:
        securityContext:
            fsGroup: 0
            runAsUser: 0
        serviceAccountName: jenkins-admin
        containers:
            - name: jenkins-agent
            image: 2534m/agent-jenkins:23
            lifecycle:
                postStart:
                exec:
                    command: ["/bin/sh", "-c", "gpasswd -a jenkins docker && sleep 5 && chmod 666 /var/run/docker.sock"]
            # resources:
            #   limits:
            #     memory: '256Mi'
            #     cpu: '500m'
            #   requests:
            #     memory: '128Mi'
            #     cpu: '250m'
            ports:
                - containerPort: 22
            volumeMounts:
                - mountPath: /var/run/docker.sock
                name: docker-sock
        volumes:
            - name: docker-sock
            hostPath:
                path: /var/run/docker.sock
    ```
1. Service
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: jenkins-agent-svc
    namespace: jenkins
    spec:
    selector:
        app: jenkins-agent
    ports:
        - port: 22
        targetPort: 22
        protocol: TCP
    ```
1. ServiceAccount
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    name: jenkins-admin
    rules:
    - apiGroups: ['']
        resources: ['*']
        verbs: ['*']
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: jenkins-admin
    namespace: jenkins
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: jenkins-admin
    roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: jenkins-admin
    subjects:
    - kind: ServiceAccount
        name: jenkins-admin
        namespace: jenkins
    ```
you can use argocd  to install agent 
![](/screenshots/argo_agent_1.png)
#### Connect agent to jenkins to run pipeline 

1. you need to define agent node  in jenkins  
The Jenkins master communicates with the agent node, directing specific tasks for execution. Utilizing agent nodes in Jenkins enables task parallelization, optimizing resource use and expediting job execution.

1. open Manage Jenkins 
2. Manage nodes and clouds
3. new node
4. enter any name 
5. enter name
6. Remote root directory  copy this (/var/jenkins_home)
> [!IMPORTANT]  
> 7. **important** Labels copy this (agent)
8.  Launch method-----------> Launch agent by connecting it to the master
9. Use WebSocket 
![](/screenshots/argo_agent_2.png)
![](/screenshots/argo_agent_3.png)
![](/screenshots/argo_agent_4.png)
10. save


```shell script
kubectl get po -n jenkins
```
```output
NAME                             READY   STATUS    RESTARTS       AGE
jenkins-0                        2/2     Running   31 (17m ago)   11d
jenkins-agent-5d5ccb8df4-d49hm   1/1     Running   4 (17m ago)    2d14h
```
now open your agent

```shell script
kubectl exec -it -n <name of agent> -- /bin/bash
```

```shell script
su jenkins
```
```shell script
curl -sO http://jenkins:8080/jnlpJars/agent.jar
java -jar agent.jar -jnlpUrl http://jenkins:8080/computer/agent/jenkins-agent.jnlp -secret 74582befe5098ba4bb9287a6e3221b98519461c949dde2dbae00e432358e4dd1 -workDir "/var/jenkins_home/"
```
![](/screenshots/argo_agent_5.png)
![](/screenshots/argo_agent_6.png)
![](/screenshots/argo_agent_7.png)

## your agent is ready

## before Create a pipeline
you need to create 3 repo
### one for our infrastructure 
```output
Continuous_Integration_infra
├── infrastructure
│   ├── all-modules
│   │   ├── dynamodb
│   │   │   ├── main.tf
│   │   │   ├── output.tf
│   │   │   └── variables.tf
│   │   ├── eks
│   │   │   ├── main.tf
│   │   │   ├── outputs.tf
│   │   │   └── variables.tf
│   │   ├── IAM
│   │   │   ├── main.tf
│   │   │   ├── outputs.tf
│   │   │   └── varibles.tf
│   │   ├── network
│   │   │   ├── elastic.tf
│   │   │   ├── internet.tf
│   │   │   ├── nat.tf
│   │   │   ├── output.tf
│   │   │   ├── rout-privet.tf
│   │   │   ├── subnet.tf
│   │   │   ├── variables.tf
│   │   │   └── vpc.tf
│   │   ├── s3
│   │   │   ├── main.tf
│   │   │   ├── outputs.tf
│   │   │   └── variables.tf
│   │   └── security
│   │       ├── output.tf
│   │       ├── securityg.tf
│   │       └── variable.tf
│   ├── demo-01
│   │   ├── dynamodb.tf
│   │   ├── provider.tf
│   │   ├── README.md
│   │   └── s3.tf
│   └── demo-02
│       ├── backend.tf
│       ├── domainssl.tf
│       ├── ekscluster.tf
│       ├── iam.tf
│       ├── network&security.tf
│       ├── provider.tf
│       ├── README.md
│       ├── terraform.tfvars
│       └── variables.tf
└── Jenkinsfile-infrastructure

```
### one for our app
```output
app
├── app
│   ├── Dockerfile
│   ├── index.html
│   ├── main.js
│   ├── styles.css
│   └── target-file.csv
├── Jenkinsfile
└── README.md

``` 
### one for  argocd-Manifest

```output
├── app_Manifest
│   ├── deployment.yml
│   └── service.yml
└── jenkins_agents_Manifest
    ├── agent_svc.yaml
    ├── agent.yaml
    └── dockerfile
```
## Create a pipeline
### Phase 1: Add your credentials
1. In the **Jenkins UI**, Click **Credentials** on the left
1. Click the **(global)** link
1. Click **Add Credentials** on the left

Credentials you need you add 
1. USER_EMAIL <---------------- USER_EMAIL for github or bitbucket
1. ECR_REPOSITORY <-----------ECR URL 
1. AWS_ACCESS_KEY_ID <---------- create AWS ACCESS KEY ID  for jenkins as user 
1. AWS_SECRET_ACCESS_KEY 
1. GIT_USERNAME 
1. GIT_PASSWORD
1. bitbuckt_jenkins <---- add it as username/password

### Phase 2:  jenkins file
#### infrastructure jenkins file
```groovy
pipeline {
  agent any
  
environment {
    AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
  }
  
  stages {
    
    stage('Terraform Init') {
      steps {
        dir('Terraform') {
          withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
            sh 'terraform init'
          }
        }
      }
    }
    
    stage('Terraform Plan') {
      steps {
        dir('Terraform') {
          withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
            sh 'terraform plan'
          }
        }
      }
    }
    
    stage('Terraform Apply') {
      steps {
        dir('Terraform') {
          withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
            sh 'terraform apply -auto-approve'
            
          }
        }
      }
    }
  }
}
```

**Stage: Terraform Init**

Purpose: This stage initializes Terraform in the specified directory (Terraform).

**Stage: Terraform Plan** 

Purpose: This stage creates an execution plan for Terraform changes.

**Stage: Terraform Apply**

Purpose: This stage applies the Terraform execution plan, making changes to the infrastructure.



#### app jenkins file

```groovy
pipeline {
  agent any

  environment {
    NAME = "argo"
    VERSION = "1.${env.BUILD_ID}"
    dockerfpath="${env.workspaceDir}/app"
    dockerfilepath="${dockerfpath}/Dockerfile"
    USER_EMAIL = credentials('USER_EMAIL')
    ECR_REPOSITORY = credentials('ECR_REPOSITORY')
    ECR_REPO = "${env.ECR_REPOSITORY}/${env.NAME}"
    IMAGE_TAG= "1.${env.BUILD_ID}"
    AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
    AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    GIT_USERNAME=credentials('GIT_USERNAME')
    GIT_PASSWORD=credentials('GIT_PASSWORD')
    workspaceDir = "${JENKINS_HOME}/workspace/${JOB_NAME}"
    GIT_APP_URL= 'https://mahmoudsamir0@bitbucket.org/mahmoudsam/continuous-integration.git'
    GIT_ARGOCD_URL= 'https://mahmoudsamir0@bitbucket.org/mahmoudsam/argocd-manifest.git'
    }
  
  stages {
    stage('pull app') {
      steps {
          script {
                  checkout([$class: 'GitSCM', 
                      branches: [[name: 'master']],
                      doGenerateSubmoduleConfigurations: false,
                      extensions: [[$class: 'CleanBeforeCheckout'], [$class: 'RelativeTargetDirectory', relativeTargetDir: env.workspaceDir ]],
                      userRemoteConfigs: [[url: env.GIT_APP_URL, credentialsId: 'bitbuckt_jenkins']]
                  ])
          }
      }
    }

    stage('Authenticate with ECR') {
      steps {
        script {
          withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')])  {
            sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REPOSITORY"
          }
        }
      }
    }
    stage('Build and Push Image') {
      steps {
sh "docker build -t ${ECR_REPOSITORY}/${NAME}:${IMAGE_TAG} -f ${dockerfilepath} ${dockerfpath}"
sh "docker push ${env.ECR_REPO}:${IMAGE_TAG}"
        }
      }

        stage('pull argocd manifest') {
            steps {
                script {
                        checkout([$class: 'GitSCM', 
                            branches: [[name: 'master']],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [[$class: 'CleanBeforeCheckout'], [$class: 'RelativeTargetDirectory', relativeTargetDir: env.workspaceDir ]],
                            userRemoteConfigs: [[url: env.GIT_ARGOCD_URL, credentialsId: 'bitbuckt_jenkins']]
                        ])
                }
            }
        }
    
      stage('Update Manifest') {
            steps {
                script {
                    def directoryPath = "${env.workspaceDir}/app_Manifest"
                    def fileName = 'deployment.yml'
                    def fullFilePath = "${directoryPath}/${fileName}"
                    sh"chmod +x ${fullFilePath}"
                    sh """
                    sed -i -E 's#${ECR_REPO}[^ ]*#${ECR_REPO}:${VERSION}#g' ${fullFilePath}
                    """
                    sh "cat ${fullFilePath}"
                      }
            }
    }

        stage('Push changes to Bitbucket') {
            steps {
                script {
                sh "git config --global user.email '${USER_EMAIL}'"
                sh "git config --global user.name '${GIT_USERNAME}'"
                sh "git add * "
                sh "git commit -m 'change image to new VERSION ${VERSION}'"
                    withCredentials([usernamePassword(credentialsId: 'bitbuckt_jenkins', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@bitbucket.org/mahmoudsam/argocd-manifest.git  HEAD:master"
                    }
                }
            }
        }
  }
}
```
### Phase 3: define environment variables

- `NAME` - ECR repo name
- `VERSION` - VERSION for docker image
- `dockerfpath` - docker file path in app dir 
- `dockerfilepath`  - docker file path in app dir and name of dockerfile 
- `USER_EMAIL` -git user email 
- `ECR_REPOSITORY`- ecr url 
- `IMAGE_TAG` - VERSION for docker image tagging 
- `AWS_ACCESS_KEY_ID` - AWS ACCESS_KEY
- `AWS_SECRET_ACCESS_KEY` - AWS SECRET ACCESS KEY
- `GIT_USERNAME` - GIT USERNAME 
- `GIT_PASSWORD` -
- `workspaceDir` GIT PASSWORD
- `GIT_APP_URL` - GIT APP URL
- `GIT_ARGOCD_URL` - GIT ARGOCD manifest URL

### Phase 4: prepare argocd

1. connect to eks 
    ```shell
    aws eks --region <your-region> update-kubeconfig --name <your-cluster-name>
    ```
    Verify that kubectl is now configured to use your EKS cluster by checking the cluster information:
    ```shell
    kubectl config get-contexts
    ```

    now add cluster to argocd 

    ```shell
    argo cluster add <cluster name >
    ```

1. connect to argocd manfest repo
open argocd 
Settings>Repositories>connectrepo

    you can choose via https or ssh 
    if you choose https you can add username and token you need to genrate in git repo 
1. apply app from argocd manifest repo

```yaml
cat << EOF >> sample-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-app
  namespace: app
spec:
  destination:
    namespace: sample-app
    server:<cluster ip>
  project: sample-project
  source:
    path: sample-app/
    repoURL: <repo url >
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
EOF
```


``` shell
kubectl apply sample-app.yaml
```
You need to prepare the app in ArgoCD to make the pipeline automated.



> [!NOTE]  
> You need to add ecr AWS ACCESS KEY ID and AWS_SECRET_ACCESS_KEY to make argocd pull image from ecr 

```shell
kubectl create secret generic argocd-ecr-access   --from-literal=AWS_ACCESS_KEY_ID=<AWS_ACCESS_KEY_ID>   --from-literal=AWS_SECRET_ACCESS_KEY=<WS_SECRET_ACCESS_KEY>   --namespace=argocd
```
### Phase 4: Create a webhook
1. bitbucket 
    1. Now it's time to create a webhook for your own repository. From your repository in Bitbucket, select Repository settings on the left sidebar, then select Webhooks.

    1. Click the Add webhook button to create a webhook for the repository.

    1. Enter Webhook Listener as the Title.

    1. Enter the jenkins URL to the server in the URL field
    1. Click Save. For the purposes of this tutorial, keep the Triggers as only a Repository Push.

1. github 
    1. Switch to your GitHub account.
    Now, go to the `Settings` option on the right corner.
    1. select the `Webhooks` option and then click on the `Add Webhook` button.
    1.   It will provide you the blank fields to add the Payload URL where you will paste your Jenkins address, Content type, and other configuration.

    1. Go to your Jenkins tab and copy the URL then paste it in the text field named `Payload URL` .

    1. The “Secret” field is optional. Let’s leave it blank for this Jenkins GitHub Webhook.

    1. Next, choose one option under  `Which events would you like to trigger this webhook?`. The 3 options will do the following events listed below:
        1. Just the Push Event: It will only send data when someone push into the repository.
        1. Send Me Everything: It will trigger, if there is any pull or push the event into the repository.
        1. Let Me Select Individual Events: You can configure for what events you want your data.
    1. Now, click on the “Add Webhook” button to save Jenkins GitHub Webhook configurations.


# Conclusion:

This CI/CD pipeline, combining ArgoCD, Amazon ECR, and Jenkins, is tailored for efficient and scalable application deployment on an Amazon EKS cluster. The project sets the foundation for a streamlined and automated development process, fostering collaboration and accelerating time-to-market for containerized applications in an EKS environment.