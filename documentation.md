# Glasswall Cloud SDK

## Architecture Overview

- Implemented as a K8s service
- Easy to deploy within ICAP Service cluster
- Interacts with Adaptation Service with RabbitMQ
- Accessible on <Cluster IP>:8080

## Architecture Diagram

![architecture](images/c-sharp-pod.png)

- The C# service receives files for a rebuild on the REST API endpoints.  
- After preliminary processing (at least must verify the file has been received) the request is passed to the `Adaption Service` with `Adaption request` RabbitMQ     message.  
- The file to be rebuilt is uploaded to the `Original Store`.  
- Once the processing is completed C# service gets informed with a RabbitMQ `Adaption outcome` message.  
- C# service get the rebuilt file from the `Rebuild Store` and passes it to the user.  

## Dataflow Diagram

![Dataflow Diagram](images/dataflow-diagram.png)

## Endpoints

| API Endpoint | Method | Description | 
|------|---------|---------    |
| /api/FileTypeDetection/base64    | POST |  /api/FileTypeDetection/base64 |
| /api/rebuild/file    | POST |  Rebuilds a file using its binary data       |
| /api/rebuild/base64   | POST | Rebuilds a file using the Base64 encoded representation |
| /api/rebuild/base64   | POST | Analyse a file using the Base64 encoded representation |

### Detailed API Endpoints Documentation - [ Link ](./ApiEndpointsDocumentation.md)

## Deployment of Glasswall Cloud SDK within ICAP Service cluster

- Make sure you can connect to the cluster with `kubectl`  
- For the case of [AMI or OVA](https://github.com/k8-proxy/glasswall-servers-eval/wiki) just SSH to the EC2 instance or the VM
- For other deployments see [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/))

- Get [deployment.yaml](https://github.com/k8-proxy/cs-k8s-api/blob/main/deployment.yaml)

    ```
        wget https://raw.githubusercontent.com/k8-proxy/cs-k8s-api/main/deployment.yaml
    ```
- Replace `<REPLACE_IMAGE_ID>` within the file with the tagged (`<dockerhub_repo>/cs-k8s-api:<version>`) docker hub image created in the previous section  

- Run the following:

    ```
        kubectl -n icap-adaptation apply -f deployment.yaml
    ```

- The command above must create a Kubernetes service called `proxy-rest-api` that can be accessed at <Cluster_IPv4>:8080  

- Verify the service has been successfully created  

    ```
        kubectl -n icap-adaptation get svc
    ```

- The output should contain a line that looks like the following:

    ```
        ...
        proxy-rest-api              LoadBalancer   10.43.236.137   91.109.25.86   8080:30329/TCP
        ...
    ```

- Verify that the service POD is running  

    ```
        kubectl -n icap-adaptation get pods
    ```
- You should see the POD in the `Running` state  

    ```
        proxy-rest-api-7b7d5b6456-s44kq                         1/1     Running     0

## Glasswall Cloud SDK with Compliant Kubernetes

### **Repository: https://github.com/k8-proxy/k8s-compliant-kubernetes/tree/cs-api**

![cs-api-ck8s Diagram](images/cs-api-ck8s-architecture.jpg)


## FileDrop with Glasswall Cloud SDK

![FileDrop-Cloud-SDK Diagram](images/filedrop_architecture.jpg)

## Integration of Glasswall Cloud SDK with FileDrop
- Clone Repo

    ```
    git clone https://github.com/k8-proxy/k8-rebuild-file-drop
    cd k8-rebuild-file-drop/app
    ```
- Edit Dockerfile 
    ```
    vim Dockerfile
    ```
- Update endpoints with c# api endpoints in below three lines
    ```
    ARG REACT_APP_ANALYSE_API_ENDPOINT='http://<ip>:<port>'
    ARG REACT_APP_FILETYPEDETECTION_API_ENDPOINT='http://<ip>:<port>'
    ARG REACT_APP_REBUILD_API_ENDPOINT='http://<ip>:<port>'
    ```
- Run : `docker build -t k8-rebuild-file-drop .`
- Run: `docker run -it --rm -p 80:80 k8-rebuild-file-drop`
- Open the `http://localhost` in your Browser
- You will be able to see File Drop, React App web interface

    ![FileDrop-UI](images/filedrop_ui.png)

- How to use file-drop [ Link ](https://github.com/k8-proxy/glasswall-servers-eval/wiki/How-to-use-File-Drop)


Note : To Deploy in AWS, an AMI needs to be created with above setup.





