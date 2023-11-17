# end_to_end_ci_cd_pipeline_Argocd

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
