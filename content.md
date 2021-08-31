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
* A ***Namespace*** is a K8's object which partitions a cluster into multiple virtual clusters.
    * The 'default' namespace is automatically set up when the cluster is created; alongside 'kube-system' (which houses the resources required for the cluster's internal working: networking, etc) and 'kube-public' (which houses resources that should be made available for all users is created).
    * Using namespaces is useful in enterprise environments where you have different versions of an application running or different applications in general (where a whole namespace is used segregate the various services that are running).
    * Namespaces can have their own RBAC (role-based access controll) and resource limits.
    * Objects that are in the same Namespace can refer to each other by their names (think Mark Smith vs Mark Williams analogy).
        * Objects in one namespaces can talk to object in another by refering the name of the object and the namespace that it is in.
            * example:
                * connect to ```svc/db-service``` in the ```default``` ns from the ```default``` ns:
                    * using '```db-service```'
                * connect to ```svc/db-service``` in the ```dev``` ns from the ```default``` ns:
                    * using '```db-service.dev.svc.cluster.local```'
    * You can specify the namespace that you want a resource to be deployed to in the definition YAML file.
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
            name: myapp
            namespace: dev
            labels:
                app: myapp
        ```
    * ```ns``` is short-hand for ```namespaces``` in ```kubectl```
* A ***ResourceQuota*** belongs to a namespace and defines the quotas that are allowed to be consumed by K8's resources in that namespace.
    * K8's Resources <-> Object; not to be confused with compute resources like CPU's, memory, etc.
* A ***ConfigMap*** is used to centrally manage configuration data for other K8s objects. 
    * Using a ConfigMap occurs in two stages:
        1. Create the ConfigMap (declaratively or imperatively [can be literal or from file])
        2. Inject it into the pod (see pod yaml below)

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
* Create a namespace (imperative)
    * ```kubectl create ns dev```
* Generate the YAML for a pod without creating it
    * ``` kubectl run redis --image=redis123 --dry-run=client -o yaml > pod.yaml ```
* List pods in a namespace
    * ```kubectl get pods -n namespace```
* List pods across all namespaces
    * ``` kuebctl get pods --all-namespaces ```
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
* Switch namespaces
    * ``` kubectl config set-context $(kubectl config current-context) -n dev ```
* Create a service for a pod (imperative)
    * ```kubectl expose redis --name redis-service --port 6379 --target-port 6379```
* Set an environment variable (imperative)
    * ```kubectl run -e APP_COLOUR=pink my-app```
* Create a ConfigMap (imperative)
    * ```kubectl create configmap <configmap-name> --from-literal=<key>=<value> \ --from-literal=<key>=<value> ...```

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
              command: ["cmd", "val"] #optional
              env: #optional
                - name: APP_COLOUR
                  value: pink
              envFrom: #optional
                - configMapRef:
                    name: <configmap-name>
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

<br>


* Namespace: <br>
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
        name: dev
    ```


<br>


* Resource Quota: <br>
    ```yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
        name: compute-quota
        namespace: dev
    spec:
        hard:
            pods: "10"
            requests.cpu: "4"
            requests.memory: "5Gi"
            limits.cpu: "10"
            limits.memory: "10Gi"
    
    ```


<br>


* ConfigMap: <br>
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: app-config
    data:
      APP_COLOUR: blue
      APP_MODE: prod
    ```

 <br>


### Docker Concepts
* Docker is a container runtime engine that is the default standard in K8s.
* Containers are not designed to run an OS, only to execute a particulary task or process.
    * A container only lives whilst the process that is running inside it is running. When it has finished the container exits.
        * ```docker run ubuntu``` runs and immeadiately exits becuase there is no process running.
        * ```docker run ubuntu sleep 5``` runs as a container for 5 seconds before exiting
            * ```docker run <image> <command>```
            * Alternatively, you can specify the command to be run in the Dockerfile:
                ```Dockerfile
                FROM ubuntu

                ENTRYPOINT ["sleep"]

                CMD ["5"]
                ```
                * this is equivalent to ```docker run ubuntu sleep 5``` but gives the option to provide a custom sleep length if we wish, i.e ```docker run ubuntu 10```.
                    * ```docker run --entrypoint sleep2.0 ubuntu 10``` lets us override the entrypoint and is equivalent to ```docker run ubuntu sleep2.0 10```
* Commands and arguments in a K8s pod
    * In the pod-definition yaml file below, the ```args``` field provides an argument to the docker container running inside the pod. This is equivalent to ```docker run ubuntu-sleeper 10```, where the ubuntu-sleeper Dockerfile has an entrypoint for the 'sleep' command.
    * The ```command``` field in the yaml below provides an alternate ```ENTRYPOINT``` to the docker container and is equivalent to ```docker run --entrypoint sleep2.0 ubuntu 10```.
    ```yaml
    apiVersion: v1 
    kind: Pod
    metadata:
        name: ubuntu-sleeper
        labels:
            app: ubuntu-sleeper
    spec:
        containers:
            - name: ubuntu-sleeper
              image: ubuntu-sleeper
              command: ["sleep2.0"]
              args: ["10"]
    ```
    * The long and short of it is: 
        ```Dockerfile 
            ...

            ENTRYPOINT ["sleep"]
            
            CMD ["5"]
        ```
        * the Dockerfile lines above are equivalent to the pod-definition yaml lines below.
        ```yaml
        ...
            containers:
            ...
            command: ["sleep2.0"]
            args: ["10"]
        ```

<br>

