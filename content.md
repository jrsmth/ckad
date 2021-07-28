# Content

<br>

### K8's Architecture

* A ***Node*** is a machine where K8's is installed and applications are run.
* A ***Cluster*** is comprised of multiple nodes. 
    * This provides high availabilty, in the event that a node fails.
    * There are two types of nodes:
        * ***Master*** nodes manage the orchestration of containers across the cluster.
        * ***Worker*** nodes handle the actual running of containerised applications.
* Components
    * Master Node
        * ***API-Server***: 
            * allows the user to interact with the cluster.
        * ***etcd***:
            * distributed keystore, stores the data used to manage the cluster.
        * ***Controller***:
            * acts to keep the running state alligned with the declared desired state.
        * ***Scheduler***:
            * looks for newly created containers and assigns them to nodes.
    * Worker Node
        * ***kubelet***:
            * agent that runs on each node, ensures containers run in pods
        * ***Container Runtime***:
            * software that runs containers, Docker
    * ***kubectl*** is the CLI that gets installed on your local environment and allows you to interact, via the API-server, with the K8's cluster.

<br>

### K8's Resources

* A ***Pod*** is the smallest object that you can create in K8's and typically is a wrapper for a single containerised application. 
    * multi-container pods are a rare use-case (side car)
        * example: a secondary container may be useful for processing user input or uploading a folder.
    * ```po``` is short-hand for ```pods``` in ```kubectl```
* A ***ReplicaSet*** gives our application high-availability by running multiple instances of our pod.
    * Even with a single pod, a ReplicaSet is used because it informs the K8's controller of how many pods are in our desired state - therefore, if our pod/s died, a new one will be spun up.
    * ReplicaSets also gives us load balancing and scaling across pods.
    * ReplicaSets have a selector field for the pods that it is responsible for - therefore, they are able to manage pods that they did not create.
        * the pod label in the replicasets' ```template``` must match the selector's ```matchLabel```
    * ***ReplicaControllers*** was the original replication object; since replaced by ReplicaSets.
    * ```rs``` is short-hand for ```replicasets``` in ```kubectl```
* A ***Deployment*** allows us to upgrade the underlying instances of our app seemlessly with rolling updates and rollbacks.
    * It is a wrapper around ReplicaSets, which are in turn, wrappers around pods.

<br>

### Imperative vs Declarative

* ***Imperative*** programming is where you directly act to create a desired state
* ***Declarative*** programming is where you indirectly act to create a desired state, by describing it.
* With K8's, we should always be declarative; so we can make use of the in-built desired state functionality and have an accurate description of what the desired state is in our definition yaml files.
    * Imperative commands, like ```create/edit pod```, should only be used in the exam, when they're explicitly required.
        * The default action should be to generate/edit the yaml and use ``` kubectl apply -f ```

<br>

### ```kubectl``` Commands

* Create a pod (imperative)
    * ```kubectl run nginx --image nginx```
* Generate the YAML for a pod without creating it
    * ``` kubectl run redis --image=redis123 --dry-run=client -o yaml > pod.yaml ```
* List pods in a namespace
    * ```kubectl get pods -n namespace```
* List all resources in a namespace
    * ```kubectl get all -n namespace```
* Detailed info about a pod
    * ```kubectl describe pod <pod-name>```
* Delete a K8's resource - example: service
    * ```kubectl delete svc/<svc-name> -n namespace```
* Create/update a K8's resource from YAML (declarative)
    * ``` kubectl create -f <definition.yaml> -n namespace ```
        * ```apply```: create + replace a resource
        * ```create```: create a new resource
        * ```replace```: update a live resource
* Edit a pod (imperative)
    * ``` kubectl edit pod <pod-name> ```
* Scale pods (imperative)
    * ``` kubectl scale --replicas=<instances> rs <pod-name> ```

<br>

### ```yaml``` Examples

* Pod: <br>
    ```yaml
    apiVersion: v1 
    kind: Pod
    metadata:
        name: nginx
        labels:
            app: nginx
    spec:
        containers:
            - name: nginx
              image: nginx
    ```

<br>

* ReplicaSet: <br>
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
        name: myapp
        labels:
            app: myapp
    spec:
        template:
            metadata:
                name: myapp
                labels:
                    app: myapp
            spec:
                containers:
                    - name: myapp
                    image: nginx
    replicas: 3
    selector: 
        matchLabels:
            app: myapp
    ```
    * the pod's ```metadata``` and ```spec``` goes under ```template```

<br>


* Deployment: <br>
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: myapp
        labels:
            app: myapp
    spec:
        template:
            metadata:
                name: myapp
                labels:
                    app: myapp
            spec:
                containers:
                    - name: myapp
                    image: nginx
    replicas: 3
    selector: 
        matchLabels:
            app: myapp
    ```
    * except for the ```Kind```, this is the same as the ReplicaSet defintion