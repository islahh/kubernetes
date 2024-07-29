# kubernetes
https://newsletter.systemdesigncodex.com/p/practical-intro-to-kubernetes?utm_campaign=post&utm_medium=web

How do Developers see Kubernetes?
For application developers working with Kubernetes, it’s typically a 3-step process:
- STEP 1 - The developers create manifest files for their applications and submit those files to Kubernetes.
- STEP 2 - Kubernetes checks those files and deploys the necessary applications to its cluster of worker nodes.
- STEP 3 - Kubernetes manages the complete lifecycle of the applications based on the manifest files.


Kubernetes is deployed as a cluster. It has two main parts:
1. The Master Node
2. Worker Nodes
   
1. Master Node
This node hosts the Kubernetes Control Plane - the entity that controls the entire cluster.
Think of it as a manager in an organization.

The control plane consists of several important parts:

- The Kube-apiserver acts as an entry point to the K8S cluster.
- A scheduler that assigns a particular deployment to a worker node
- Controller Manager for tracking the nodes and handling failures
- etcd for storing the cluster’s configuration

2. Worker Nodes
There are multiple worker nodes controlled by a single master node. They are the infrastructure pieces that host the containerized applications.
Think of them as employees under a manager.

A worker node is made up of the following components:

- A container runtime such as Docker or rkt.
- Kubelet for talking to the API server and managing containers. Basically, the kubelet makes sure that the node is operating according to the configuration.
- kube-proxy for communication of pods within the cluster and to the outside network.

**Top Kubernetes Resources**
### 1. pods
Pods are the smallest atomic unit that you create in Kubernetes.

Aren’t pods the same as a container?
Not exactly.

Kubernetes groups multiple containers into a single unit known as a Pod. These pods are deployed on the worker nodes we talked about earlier.

`
While multi-container pods have their uses, the common consensus is to have 1 container per pod unless needed otherwise.

This approach makes it easy to scale an application without worrying about multiple processes.
`

Here’s a manifest file to declare a very simple pod.

```Bash apiVersion: v1
kind: Pod
metadata:
   name: basic-pod-demo
spec:
   containers:
   - image: systemdesigncodex/nodejs-demo
     name: hello-service
```

**Some important points to note over here**
- The kind is the type of resource.
- In the metadata section, the name field contains the name of the pod.
- Within the spec section, you can provide details about the containers that are going to be part of the pod. In this example, there’s only one container using the image named systemdesigncodex/nodejs-demo.
- Lastly, we have the name of the container.

To create a Kubernetes resource in your cluster, you use the kubectl command. Once the resource (such as pod is applied), use kubectl to check the status.

```Bash
$ kubectl apply -f <file-name>.yaml
$ kubectl get po
``` 

### 2. service
Kubernetes is ideal for managing microservices, deploying one microservice per pod, and ensuring they can communicate seamlessly. Pods, the smallest deployable units in Kubernetes, often need to interact with each other, necessitating a mechanism for pods to locate and connect with each other within the cluster. Kubernetes Services are pivotal in addressing this need.
![image](https://github.com/user-attachments/assets/f3c6aa2b-ab92-48de-84ef-6075dd264e0d)

Here’s the YAML file to create a simple Kubernetes Service resource:

```Bash
apiVersion: v1
kind: Service
metadata:
   name: basic-service
spec:
   ports:
   - port: 80
     targetPort: 3000
   selector:
     app: hello-service
```
The port mapping is important over here:

- The port attribute is the service port
- The targetPort is the port exposed by your application container. For example, a webserver running on port 3000.

### 3. Deployment
A Kubernetes Deployment is a higher-level resource you can use for deploying applications and updating them declaratively.
When you create a Deployment, a ReplicaSet resource is also created under the hood to control the pods.

What does a ReplicaSet do?
It lets you create multiple pods for the same application. Think about horizontally scaling your application.
A Deployment sits on top of the ReplicaSet, making sure that the latest version of your app is always running on the desired number of pods.

![image](https://github.com/user-attachments/assets/bf4d42a5-3bf5-44a6-a806-c7ecfc16a9d7)

Also, here’s an example for the Deployment YAML file.

```Bash
apiVersion: apps/v1
kind: Deployment
metadata:
   labels:
     app: nginx
   name: nginx
spec:
   replicas: 3
   selector:
     matchLabels:
       app: nginx
   template:
     metadata:
       labels:
         app: nginx
     spec:
       containers:
       - image: progressivecoder/nginx
         name: nginx
         ports:
         - containerPort: 80
```
Notice the value of the replicas parameter in the spec section. 
It tells Kubernetes how many pods need to be created for the nginx container.

Note that if you deploy pods using a Deployment, you don’t need to create separate YAML files for pods. 
Neither do you need to create a separate ReplicaSet resource.

### 4. Volumes
Kubernetes volumes are a component of a pod and not a standalone object.

But what is the use of volumes?

Think of them as a form of storage.

In certain scenarios, you may want the application to persist some data to the disk.

Check the below diagram that shows the concept of a Kubernetes Volume.

![image](https://github.com/user-attachments/assets/2b746b97-3a33-45fe-a0f6-4c3ad857d8a4)

So - how to define a Kubernetes Volume?

You declare them as part of the Pod YAML or within the Deployment YAML.

See the below example where we add Volume to the Deployment YAML:
```Bash
apiVersion: apps/v1
kind: Deployment
metadata:
   labels:
     app: nginx
   name: nginx
spec:
   replicas: 1
   selector:
     matchLabels:
      app: nginx
   template:
     metadata:
      labels:
       app: nginx
     spec:
       containers:
       - image: systemdesigncodex/sidecar
         name: sidecar
         env:
         - name: STATIC_SOURCE
           value: https://raw.githubusercontent.com/dashsaurabh/sidecar-demo/master/index.html
         volumeMounts:
         - name: shared-data
           mountPath: /usr/share/nginx/html
       - image: systemdesigncodex/nginx
         name: nginx
         ports:
         - containerPort: 80
         volumeMounts:
         - name: shared-data
           mountPath: /usr/share/nginx/html/
       volumes:
       - name: shared-data
         emptyDir: {}
```
Some important points about the above YAML:

- The volumes section at the very bottom declares a volume of type emptyDir. What it means is that volume starts as an empty directory. It is given the name “shared-data”
- The app running inside the pod can write any files it needs to the empty directory.
- In the above example, the emptyDir volume is used for sharing files between the “sidecar” container and the “nginx” container.
- Within the containers section, the volume “shared-data” is mounted at /usr/share/nginx/html using the volumeMounts section. The mounting is done for both the containers enabling them to share the data.
- When the pod is destroyed, the volume is also destroyed along with it.

## NEXT

ingress: 
container?


## Side and Future Notes
### Helm Charts

Helm charts are a collection of files that describe a Kubernetes cluster’s resources and package them together as an application. They comprise three basic components: 

The chart - Chart.yaml defines the application metadata like name, version, dependencies, etc. 
Values - values.yaml sets values, which is how you will set variable substitutions for reusing your chart
You may also have a values JSON schema that describes a structure for the values file, which can help in creating dynamic forms and validating your values parameters.
The templates directory - templates/ houses your templates and combines them with the values set in your values.yaml file to create manifests
The charts directory - charts/ stores any chart dependencies you define in Chart.yaml and reconstruct with helm dependency build or helm dependency update.
Each time you install a Helm chart, you also create an instance of it, called a release. Helm charts are maintained with each new release, and you can easily use previous versions of the chart to roll back to your preferred configuration.
