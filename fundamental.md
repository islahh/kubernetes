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
<details><summary>Pods</summary>
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
</details>

<details><summary>Service</summary>
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
</details>
   
<details><summary>Deployment</summary>
   
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
</details> 

<details><summary>Volumes</summary>
   
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
</details>
<details>
  <summary>Kubernetes Namespaces</summary>

 In Kubernetes, a namespace is a way to divide cluster resources into virtual partitions. It's primarily used to create separate environments within a single Kubernetes cluster, allowing teams or projects to share the same physical cluster securely. Here are key aspects of namespaces in Kubernetes:

Logical Partitioning: Namespaces provide a way to logically divide cluster resources, such as pods, services, and replication controllers, into distinct groups. This separation helps in organizing and managing resources and enables multi-tenancy within a cluster.

Isolation and Scope: Each namespace provides a scoped environment where resources within one namespace are isolated from resources in another namespace. This isolation helps prevent naming conflicts and resource collisions between different teams or projects using the same cluster.

Default Namespace: Kubernetes clusters come with a default namespace called default. If resources are created without specifying a namespace, they are automatically assigned to this default namespace.

Managing Resources: Resources within a namespace can reference each other directly by their names, without needing to specify the namespace explicitly. For example, a service within a namespace can reference a pod by its name within the same namespace.

Security and Access Control: Kubernetes uses namespaces for access control and resource quota management. Role-based access control (RBAC) policies can be applied per namespace, allowing fine-grained control over who can access or modify resources within that namespace.

Scoping Network Policies: Network policies can be scoped to namespaces, defining rules for inbound and outbound traffic to and from pods within the same namespace. This helps enforce security and communication policies at the namespace level.

Resource Quotas: Quotas can be applied per namespace to limit the amount of compute resources (CPU, memory) and storage that can be consumed by resources within that namespace. This prevents one namespace from consuming all cluster resources and affecting others.

Overall, namespaces in Kubernetes provide a powerful mechanism for managing and organizing cluster resources, promoting isolation, security, and efficient resource utilization within a shared Kubernetes environment

In Summary
- Namespace provides a way to divide cluster resources into separate environments.
- Namespace is useful for organizing resources, managing permissions, and avoiding naming conflicts.
### NOTES FROM LINKEDIN COURSE
- Namespaces in Kubernetes: They help isolate and organize workloads, allowing you to separate applications and microservices in different environments like development and production.
- Creating a Namespace: You can create a namespace using a YAML manifest and the kubectl apply -f namespace.yaml command.
- Managing Namespaces: Use kubectl get namespaces to list namespaces and kubectl delete -f namespace.yaml to delete them.
</details>

## NEXT

ingress: 
<details>
   <summary>Ingress</summary>
- Manages external access to services within a cluster, typically HTTP.
- Provides load balancing, SSL termination, and name-based virtual hosting.
</details>


<details>
   <summary>ConfigMap</summary>
Configuration in Kubernetes:

Traditional Methods: Just like on a native host, you can configure software using command line arguments, environment variables, and config files.
Kubernetes' Opinion: Kubernetes encourages separating code from its configuration. This is reflected in its design and helps in managing configurations more effectively.

ConfigMaps:

What is a ConfigMap?: A ConfigMap is a Kubernetes object used to store configuration data in key-value pairs. This separates the configuration from the application code.
Structure: A ConfigMap has an apiVersion, kind, and metadata section, but it doesn't have a spec section. Instead, it goes straight to data, which contains the key-value pairs.

Using ConfigMaps:

Environment Variables: You can pass configuration data to your pods using environment variables. For example, you can override the color of a container by setting an environment variable using a ConfigMap.
Config Files: ConfigMaps can also store entire configuration files. You can create a ConfigMap from a file and then mount it as a volume in your pod, making the file available to your application.


Example Breakdown:
Environment Variable Example:

ConfigMap Definition:
yaml
``` bash
apiVersion: v1
kind: ConfigMap
metadata:
name: color-config
data:
colour: pink
```

Pod Definition:
yaml
``` bash
env:
name: COLOUR
valueFrom:
configMapKeyRef:
name: color-config
key: colour
```
Outcome: When the pod runs, it uses the colour value from the ConfigMap, setting the environment variable COLOUR to pink.


Config File Example:

Creating a ConfigMap from a File:
bash
kubectl create configmap website --from-file=index.html

Pod Definition:
yaml
``` bash
volumes:
name: website-volume
configMap:
name: website
volumeMounts:
mountPath: /usr/share/nginx/html
name: website-volume
```
Outcome: The ConfigMap website is mounted as a volume, replacing the default Nginx home page with the custom index.html file.



Why Use ConfigMaps?
Separation of Concerns: Keeping configuration separate from code makes it easier to manage and update configurations without changing the application code.
Flexibility: ConfigMaps allow you to change configurations dynamically, making your applications more flexible and easier to manage.
</details>


## Key Terms and Concepts

1. Cloud Native?
cloud native technologies are open-source projects designed to let technologists use cloud computing services to automatically deploy and scale applications.
Kubernetes is on example of cloud native technology.

## Side and Future Notes
<details>
  <summary>## Helm Charts</summary>
### Helm Charts

Helm charts are a collection of files that describe a Kubernetes cluster’s resources and package them together as an application. They comprise three basic components: 

- The chart - Chart.yaml defines the application metadata like name, version, dependencies, etc.
- Values - values.yaml sets values, which is how you will set variable substitutions for reusing your chart
-    You may also have a values JSON schema that describes a structure for the values file, which can help in creating dynamic forms and validating your values parameters.
- The templates directory - templates/ houses your templates and combines them with the values set in your values.yaml file to create manifests
- The charts directory - charts/ stores any chart dependencies you define in Chart.yaml and reconstruct with helm dependency build or helm dependency update.
Each time you install a Helm chart, you also create an instance of it, called a release. Helm charts are maintained with each new release, and you can easily use previous versions of the chart to roll back to your preferred configuration.

Explosion of YAML: As you scale Kubernetes, you'll encounter a large number of YAML files, making it challenging to manage and maintain them.
DRY Principle: The "Don't Repeat Yourself" (DRY) principle is crucial in software engineering to avoid redundancy. Tools like Helm, Jasonette, and Kustomize help manage and template YAML files to adhere to this principle.
Helm and Kustomize: Helm is a popular tool for templating and managing YAML files, while Kustomize combines literal YAMLs with patches, gaining traction for its unique approach.

These tools and principles are essential for maintaining efficient and manageable Kubernetes deployments.

Helm Charts:

What They Are: Helm charts are packages of pre-configured Kubernetes resources.
How They Work: Think of a Helm chart as a blueprint for deploying an application. It contains templates that generate the YAML files needed to deploy your application in Kubernetes.

Templates and Values:

Templates: These are like the skeleton of your deployment files. They have placeholders that get filled in with actual values.
Values Files: These files provide the actual values for the placeholders in your templates. You can have different values files for different environments (e.g., development and production).

Example Scenario:

Development vs. Production: Suppose you have a service that you want to deploy in both development and production environments. The core service is the same, but there are slight differences, like domain names and whether TLS (Transport Layer Security) is enabled.
Using Helm: You create a Helm chart with templates for your service. You then create values files for each environment. For example, the development values file might set the domain to dev.example.com and disable TLS, while the production values file sets the domain to prod.example.com and enables TLS.

Commands:

helm template: This command processes your templates and values files to generate the final YAML files. You can use this to preview what will be deployed.
helm install: This command deploys your application to the Kubernetes cluster using the generated YAML files.
helm upgrade: This command updates an existing deployment with new changes. Helm tracks the history of deployments, so you can roll back if something goes wrong.


Why Use Helm?
DRY Principle: Helm helps you avoid repeating yourself by using templates and values files. This makes it easier to manage and update your deployments.
Consistency: By using Helm, you ensure that your deployments are consistent across different environments.
Ease of Management: Helm tracks the history of your deployments, making it easier to manage updates and rollbacks.

Analogy:
Think of Helm like a recipe book for cooking. The templates are the recipes, and the values files are the specific ingredients you use for different occasions. Whether you're cooking for a small family dinner (development) or a large banquet (production), the core recipe remains the same, but the ingredients might vary slightly.

</details>

