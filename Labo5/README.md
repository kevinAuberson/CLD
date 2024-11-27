# Lab 05: Kubernetes

## Organisation

- Duration of this lab is 4 periods.

## Pedagogical objectives

- Become familiar with Kubernetes and its command-line tool `kubectl`
    
- Define, deploy, run and scale a web-application using the Kubernetes abstractions of Pods, Services and Deployments
    
- Use a local cluster for a test deployment before deploying on a public cloud service
    
- Verify elasticity and resilience of a Kubernetes-managed application
    

## Introduction and prerequisites

In this lab you will perform a number of tasks and document your progress in a lab report. Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

You will first install a Kubernetes test cluster on your local machine using the `minikube` tool. You manage this cluster through the `kubectl` command line tool. Then you will deploy a provided “To Do” reminder application that uses a Redis key-value store as example. The goal of the first part is to deploy a complete version of a three-tier application (Frontend + API Server + Redis) using Pods.

In the second part of the lab you will deploy the same application on a public cloud service for running Kubernetes containers: Google Kubernetes Engine.

The third part of the lab will require you to make the application resilient to failures. You will deploy multiple Pods with a Deployment for the Frontend and API services.

The following resources and tools are required for this laboratory session:

- Any modern web browser
- The Minikube command line tool (instructions to install below in Task 1)
- The Kubernetes command line tool `kubectl` (instructions to install below in Task 1)
- A Google Cloud platform account

## Task 1 - Deploy the application on a local test cluster

In this task you will set up a local Kubernetes test cluster and deploy an example application to it.

![Architecture of ToDo app.](https://cyberlearn.hes-so.ch/pluginfile.php/2139867/mod_assign/introattachment/0/todoapp.png)  

todoapp

The goal of the provided example application is to store and edit a To Do list. The application consists of a simple Web UI (Frontend) that uses a REST API (API service) to access a Key-Value storage service (Redis). Ports to be used for Pods and Services are shown in the figure.

The required Docker images are provided on [Docker Hub](https://hub.docker.com/):

- Frontend: [icclabcna/ccp2-k8s-todo-frontend](https://hub.docker.com/r/icclabcna/ccp2-k8s-todo-frontend)
- API backend: [icclabcna/ccp2-k8s-todo-api](https://hub.docker.com/r/icclabcna/ccp2-k8s-todo-api)
- Redis 3.2.11-alpine: [redis:3.2.11-alpine](https://hub.docker.com/layers/library/redis/3.2.11-alpine/images/sha256-ca0b6709748d024a67c502558ea88dc8a1f8a858d380f5ddafa1504126a3b018?context=explore)

### Subtask 1.1 - Installation of Minikube

[Minikube](https://github.com/kubernetes/minikube) is a tool that makes it easy to run Kubernetes locally. Minikube creates a single-node Kubernetes cluster inside a virtual machine on your local machine.

Minikube requires a container or virtual machine manager, such as Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare.

Install Minikube by following the instructions in [minikube start](https://minikube.sigs.k8s.io/docs/start/).

**Caution:** After installation, Minikube may tell you that you have an old version of `kubectl` installed and offer to download the newest version. For example, if you have the Google Cloud SDK, it already installed `kubectl`. In this case it is better to update via the tool that installed it: `gcloud components update`. Same applies if you have installed `kubectl` via a package manager such as Homebrew, Chocolatey or Scoop.

Verify successful installation by running `minikube version`. You should see output similar to

```
$ minikube version: v1.25.2
```

### Subtask 1.2 - Installation of kubectl

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) is a command line tool to control Kubernetes clusters.

If you have installed the Google Cloud SDK you already have `kubectl` (in this case do a `gcloud components update` to get the latest version).

If you don’t have `kubectl` you can install it through `minikube`:

```
$ minikube kubectl -- get po -A
```

Verify successful installation by running `kubectl version`. You should see something similar to

```
Client Version: version.Info{Major:"1", Minor:"17+", [...]
Server Version: version.Info{Major:"1", Minor:"23", [...]
```

Note that _Client Version_ is the version of `kubectl` while _Server Version_ refers to the version of the Kubernetes cluster. The two version numbers should be close, but it is OK if they are not the same.

### Subtask 1.3 - Create a one-node cluster on your local machine

Create and start a one-node cluster.

NB: If your machine has an ARM processor add to the `start` command the option `-–driver=docker` so that minikube runs Kubernetes in a Docker container instead of a VirtualBox VM.

Run

```
$ minikube start
minikube v1.25.2 on Darwin 12.3.1
Automatically selected the docker driver. Other choices: hyperkit, virtualbox, ssh
Starting control plane node minikube in cluster minikube
Pulling base image ...
Downloading Kubernetes v1.23.3 preload ...
[...]
Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

This will download a VM image for the Kubernetes cluster and start it. Also the tool `kubectl` is configured to use this cluster by default.

Examine the cluster:

```
$ kubectl cluster-info
```

You should see a running _Kubernetes master (Control Plane)_.

To view the nodes in the cluster, run the `kubectl get nodes` command:

```
$ kubectl get nodes
```

This command shows all nodes that can be used to host our applications. Now we have only one node, and we can see that its status is **Ready** (it is ready to accept applications for deployment).

Detailed info on the available `kubectl` commands, syntax and parameters can be found in the `kubectl` cheat sheet: [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

#### Optional: Access the Kubernetes Dashboard

The [Kubernetes Dashboard](https://github.com/kubernetes/dashboard/) gives you a graphical web interface to your Kubernetes cluster. It is itself bundled as a Pod, therefore it runs on the cluster as any other Pod (although in a separate namespace). The Dashboard is already included with Minikube.

To access Dashboard, open a separate terminal and run:

```
$ minikube dashboard
```

This will enable Dashboard and open it in the default browser.

### Subtask 1.4 - Deploy the application

Next you create the configuration and deploy the three tiers of the application to the Kubernetes cluster.

#### Deploy the Redis Service and Pod

Use the following commands to deploy Redis using the provided configuration files:

```
$ kubectl create -f redis-svc.yaml

$ kubectl create -f redis-pod.yaml
```

and verify it is up and running and on which ports using the command `kubectl get all`.

To zoom in on a Kubernetes object and see much more detail try `kubectl describe pod/redis` for the Pod and `kubectl describe svc/redis-svc` for the Service.

#### Deploy the ToDo-API Service and Pod

Using the `redis-svc.yaml` file as example and information from `api-pod.yaml`, create the `api-svc.yaml` configuration file for the API Service. The Service has to expose port 8081 and connect to the port of the API Pod.

Be careful with the indentation of the YAML files. If your code editor has a YAML mode, enable it.

- Deploy and verify the API-Service and Pod (similar to the Redis ones) and verify that they are up and running on the correct ports.

#### Deploy the Frontend Pod

Using the `api-pod.yaml` file as an example, create the `frontend-pod.yaml` configuration file that starts the UI Docker container in a Pod.

- Docker image for frontend container on Docker Hub is [icclabcna/ccp2-k8s-todo-frontend](https://hub.docker.com/r/icclabcna/ccp2-k8s-todo-frontend)

Note that the container runs on port: 8080

It also needs to be initialized with the following environment variables (check how `api-pod.yaml` defines environment variables):

- `API_ENDPOINT_URL`: URL where the API can be accessed e.g., [http://localhost:9000](http://localhost:9000/)
    - _What value must be set for this URL ?_

Hint: remember that anything you define as a Service will be assigned a DOMAIN that is visible via DNS everywhere in the cluster and a PORT.

- Deploy the Pod using `kubectl`.

#### Verify the ToDo application

Now you can verify the the ToDo application is working correctly by connecting to the Frontend Pod. As the Pod’s IP address is only accessible from inside the cluster, we have to use a trick.

The `kubectl` program is able to establish a secure tunnel between your local machine and the cluster. It listens on a port on your local machine and as soon as someone (e.g., your browser) establishes a TCP connection to that port, `kubectl` forwards the connection to the cluster. This is called _port forwarding_. There is one limitation, though: the TCP connection can only be forwarded to a Pod.

To start port forwarding, run

```
$ kubectl port-forward pod_name local_port:pod_port
```

where `pod_name` is the name of the Frontend Pod, `pod_port` is the port where the Frontend Pod is listening, and `local_port` is a free port on your local machine, say 8001.

The command blocks and keeps running to keep the tunnel open. Using your browser or another terminal window you can now access the Pod at [http://localhost:local_port](http://localhost:local_port/).

You should see the application’s main page titled **Todos V2** and you should be able to create a new To Do item. Be patient, the application will be very slow at first.

#### Deliverables

Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

#### Troubleshooting

Several things can be misconfigured. Remember that there are two Service dependencies:

- the Frontend forwarding requests to the (not externally accessible) API Service;
- the API Service accessing the Redis Service (also only accessible from within the cluster).

##### Consulting the Pod logs

You can look at the Pod logs to find out where the problem is. You can see the logs of a Pod using:

```
$ kubectl logs -f pod_name
```

##### Connecting to a Pod

You may want to test if a Pod is responding correctly on its port. Use `kubectl port-forward` as described at the end of task 1.

##### Running a command in a container

A handy way to debug is to log in to a container and see whether the required services are visible to the container.

You can run a command on a (container in a) Pod using:

```
$ kubectl exec -it pod_name <command>
```

E.g. use `kubectl exec -it pod-name bash` to start a Bash shell inside the container.

Container images tend to contain only the strict minimum to run the application. If you are missing a tool such as `curl` you can install it (assuming a Debian-family distribution):

```
$ apt-get update

$ apt-get install curl
```

## Task 2 - Deploy the application in Kubernetes Engine

In this task you will deploy the application in the public cloud service Google Kubernetes Engine (GKE).

### Subtask 2.1 - Create Project

Log in to the Google Cloud console at [http://console.cloud.google.com/](http://console.cloud.google.com/). Navigate to the **Resource Manager** ([https://console.cloud.google.com/cloud-resource-manager](https://console.cloud.google.com/cloud-resource-manager)) and create a new project.

### Subtask 2.2 - Create a cluster

**UPDATE 30.04.2024 16:10 :**

- When creating a GKE cluster, the Google Cloud Console tries to create an "Autopilot" cluster. Please find the button to create a "Standard" cluster.
- When setting the instance type as described below, please reduce the disk size from 100Go to 10Go or you won't be able to create your cluster.

Go to the Google Kubernetes Engine (GKE) console ([https://console.cloud.google.com/kubernetes/](https://console.cloud.google.com/kubernetes/)). If necessary, enable the Kubernetes Engine API. Then create a cluster.

- Choose a **GKE Standard** cluster.
- Give it a **name** of the form _gke-cluster-1_
- Select a **region** close to you.
- Set the **number of nodes** to 2.
- Set the **instance type** to micro instances.
- Keep the other settings at their default values.

### Subtask 2.3 - Deploy the application on the cluster

Once the cluster is created, the GKE console will show a **Connect** button next to the cluster in the cluster list. Click on it. A dialog will appear with a command-line command. Copy/paste the command and execute it on your local machine. This will download the configuration info of the cluster to your local machine (this is known as a _context_). It also changes the current context of your `kubectl` tool to the new cluster.

To see the available contexts, type

```
$ kubectl config get-contexts
```

You should see two contexts, one for the Minikube cluster and one for the GKE cluster. The current context has a star `*` in front of it. The `kubectl` commands that you type from now on will go to the cluster of the current context.

With that you can use `kubectl` to manage your GKE cluster just as you did in task 1. Repeat the application deployment steps of task 1 on your GKE cluster.

Should you want to switch contexts, use

```
$ kubectl config use-context <context>
```

### Subtask 2.4 - Deploy the ToDo-Frontend Service

On the Minikube cluster we did not have the possibility to expose a service on an external port, that is why we did not create a Service for the Frontend. Now, with the GKE cluster, we are able to do that.

Using the `redis-svc.yaml` file as an example, create the `frontend-svc.yaml` configuration file for the Frontend Service.

Unlike the Redis and API Services the Frontend needs to be accessible from outside the Kubernetes cluster as a regular web server on port 80.

- We need to change a configuration parameter. Our cluster runs on the GKE cloud and we want to use a GKE load balancer to expose our service.
- Read the section “Publishing Services - Service types” of the K8s documentation [https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- Deploy the Service using `kubectl`.

This will trigger the creation of a load balancer on GKE. This might take some minutes. You can monitor the creation of the load balancer using `kubectl describe`.

#### Verify the ToDo application

Now you can verify if the ToDo application is working correctly.

- Find out the public URL of the Frontend Service load balancer using `kubectl describe`.
- Access the public URL of the Service with a browser. You should be able to access the complete application and create a new ToDo.

### Deliverables

Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report (if they are unchanged from the previous task just say so).

Take a screenshot of the cluster details from the GKE console. Copy the output of the `kubectl describe` command to describe your load balancer once completely initialized.

## Task 3 - Add and exercise resilience

By now you should have understood the general principle of configuring, running and accessing applications in Kubernetes. However, the above application has no support for resilience. If a container (resp. Pod) dies, it stops working. Next, we add some resilience to the application.

### Subtask 3.1 - Add Deployments

In this task you will create Deployments that will spawn Replica Sets as health-management components.

Converting a Pod to be managed by a Deployment is quite simple.

- Have a look at an example of a Deployment described here: [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- Create Deployment versions of your application configurations (e.g. `redis-deploy.yaml` instead of `redis-pod.yaml`) and modify/extend them to contain the required Deployment parameters.
- Again, be careful with the YAML indentation!
- Make sure to have always 2 instances of the API and Frontend running.
- Use only 1 instance for the Redis-Server. Why?
- Delete all application Pods (using `kubectl delete pod ...`) and replace them with deployment versions.
- Verify that the application is still working and the Replica Sets are in place. (`kubectl get all`, `kubectl get pods`, `kubectl describe ...`)

### Subtask 3.2 - Verify the functionality of the Replica Sets

In this subtask you will intentionally kill (delete) Pods and verify that the application keeps working and the Replica Set is doing its task.

Hint: You can monitor the status of a resource by adding the `--watch` option to the `get` command. To watch a single resource:

```
$ kubectl get <resource-name> --watch
```

To watch all resources of a certain type, for example all Pods:

```
$ kubectl get pods --watch
```

You may also use `kubectl get all` repeatedly to see a list of all resources. You should also verify if the application stays available by continuously reloading your browser window.

- What happens if you delete a Frontend or API Pod? How long does it take for the system to react?
- What happens when you delete the Redis Pod?
- How can you change the number of instances temporarily to 3? Hint: look for scaling in the deployment documentation
- What autoscaling features are available? Which metrics are used?
- How can you update a component? (see “Updating a Deployment” in the deployment documentation)

### Subtask 3.3 - Put autoscaling in place and load-test it

On the GKE cluster deploy autoscaling on the Frontend with a target CPU utilization of 30% and number of replicas between 1 and 4.

Load-test using Vegeta (500 requests is enough).

##### Notes:

- The autoscale can take a while to trigger.
    
- If your autoscaling fails to get the cpu utilization metrics, run the following command
    
    - ```
          $ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
        ```
        
    - Then add the _resources_ part in the _container part_ in your `frontend-deploy` :
    - `yaml spec: containers: - ... env: - ... resources: requests: cpu: 10m`

#### Deliverables

Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report.

## Task 4 - Cleanup

At the end of the lab session:

- Delete the GKE cluster
    
- Delete the project
    
- If you want to remove the Minikube Virtual Machine from your local machine, run
    

```
$ minikube stop

$ minikube delete
```

## Additional Documentation

Kubernetes documentation can be found on the following pages:

- User Guide explaining the main concepts: [https://kubernetes.io/docs/user-guide/](https://kubernetes.io/docs/user-guide/)
- What is a Pod: [https://kubernetes.io/docs/concepts/workloads/pods/pod/](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
- What is a Service: [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)
- What is a Deployment: [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- HowTo define environment variables: [https://kubernetes.io/docs/tasks/configure-pod-container/define-environment-variable-container/](https://kubernetes.io/docs/tasks/configure-pod-container/define-environment-variable-container/)
- How-To’s for do typical tasks in Kubernetes: https://kubernetes.io/docs/tasks/
- Interactive `kubectl` command reference: [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- How to fix failed get resource metric in Kubernetes HPA: [https://aptakube.com/blog/how-to-fix-failedgeteesourcemetric-hpa](https://aptakube.com/blog/how-to-fix-failedgeteesourcemetric-hpa)

## Troubleshooting

### Display the log of a Pod

See the log of a Pod [with streaming enabled]:

```
$ kubectl logs [-f] pod_name
```

### Run a shell session inside a container

Run an interactive shell command in a Pod:

```
$ kubectl exec -it pod_name -- sh
```

