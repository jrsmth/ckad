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

### K8's Objects

* A ***Pod*** is the smallest object that you can create in K8's and typically is a wrapper for a single containerised application. 
    * multi-container pods are a rare use-case (side car)
        * example: a secondary container may be useful for processing user input or uploading a folder.
    * notes:
        * ```po``` is short-hand for pods in ```kubectl```



<br>

### ```kubectl``` Commands

* Create a pod (imperative)
    * ```kubectl run nginx --image nginx```
* List pods in namespace
    * ```kubectl get pods -n namespace```
* Detailed info about a pod
    * ```kubectl describe pod <pod-name>```



