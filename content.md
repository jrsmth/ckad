# Content

<br>

### What is Kubernetes?

* ***Kubernetes*** (K8s) is an open-source container orchestation platform for automatic deployment, scaling and management of containerised applications.

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
    * ```cm``` is short-hand for ```configmaps``` in ```kubectl```
* A ***Secret*** is used to store sensitive information
    * They are similar to ConfigMaps but the info is stored in an encoded format.
    * Secrets have no shorthand in ```kubectl```
    * Secrets, by themselves, are not very secure
        * Secrets are encoded in Base64 so anyone can decode them into plain text.
        * It is the user's practises around Secrets that make them secure; as well as the K8s architecture.
            * user: don't check secret defintion files into code repositories
            * k8s: secrets stored in a tmp fs and not written to disk; secret is only given to a pod that needs it; when a pod that requires a secret dies and leave the secret without a pod relying on it, the kubelet deletes the local copy of the secret.
* A ***Service Account*** is an account used by machines; a user account is used by humans. An example use of a SA, would be for Prometheous: which would use a SA to interact with the K8s cluster to derrive performance metrics. Additionally, Jenkins would use a SA to deploy to the cluster.
    * SA's can be created using ```kubectl```
    * When the SA is created, a token is generated. It is this token that an external application uses to authenticate with the K8s cluster.
        * The token is stored as a Secret.
        * To view the token, run ```kubectl describe secret/<secret-name>```
        * The token can then be used as an authentication bearer token when you make a REST call to the K8s API.
        * If the 3rd-party app is actually running on the K8s cluster, you can mount the Secret as a volume inside the pod running this application. S
            * Specify the SA in the pod yaml. If the pod was running you must delete it and then make the edit, as the SA field cannot be changed at runtime.
    * For every ns on the cluster, there exists a distinct ```default``` SA.
        * Every pod in the ns, when created, has the default SA and its token mounted as a volume mount. This can be used by 3rd party apps but is heavily restricted.


<br>

### Imperative vs Declarative

* ***Imperative*** programming is where you directly act to create a desired state
* ***Declarative*** programming is where you indirectly act to create a desired state, by describing it.
* With K8's, we should always be declarative; so we can make use of the in-built desired state functionality and have an accurate description of what the desired state is in our definition yaml files.
    * Imperative commands, like ```create/edit pod```, should only be used in the exam, when they're explicitly required.
        * The default action should be to generate/edit the yaml and use ``` kubectl apply -f ```

<br>

### Taints, Tolerations, Node Selectors and Node Affinity
* Taints and Tolerations are used to restrict nodes from scheduling certain pods. The reverse is not true: i.e they do not tell pods to go to certain nodes, only which nodes they cannot go to -> i.e they do not gaurantee that pods will be scheduled on a particular node.
    * A ***Taint*** is applied to a node to *repel* pods that are intolerant to the taint.
        * By default, a pod is intolerant of all taints and so will not be scheduled on a node with any taint, unless it has the correct toleration.
    * A ***Toleration*** is applied to a pod that allows it to overcome a taint.
* Master nodes have a taint that prevents any pods from being scheduled on it.
* A ***Node Selector*** works in the opposite way: they allow a pod to choose which node to be scheduled on. 
    * You assign a key value label to the node and reference this as a node selector field in the pod definition.
    * The label must be assigned to the node before you create the pod and can be done using the ```kubectl label node <node-name> <key>=<value> ```
    * Node selectors do not allow you to create complex node selection rules like OR or NOT and instead, Node Affinity must be used.
* ***Node Affinity*** gives us advanced features to choose which node we want to schedule our pod on.
    * There are two types of node affinity:
        * *required during scheduling, ignored during execution*
            * use when pod placement is crucial - if no matching node is found, pod is not scheduled
        * *prefered during scheduling, ignored during execution*
            * use for most cases, as pod will still be scheduled on another (nom-matching) node
        * *required during scheduling, required during execution***
            * will not schedule pods if a matching node is not found and will evict live pods that break affinity rules (useful if the rules change)
* Tying it together:
    * Let's say you can a red pod, a green pod and a blue pod; as well as a red node, a green node and a blue node. These there pods all belong to the same application and you want the colors to match. In addition, there are two grey (different application) pods and two grey (different application) nodes on the same cluster. We want to ensure that the green pod is only deployed on the green node (etc) and that no grey pods make it onto our application nodes; likewise no coloured pods end up on the grey nodes.
        * To do this, we need a combination of Taints, Tolerations and Node Affinity:
            * Apply an application-wide taint to the red, green and blue nodes so that only pods that that have the application-wide toleration will be scheduled on it. 
                * Apply this toleration to the red, blue and green pods. 
                * If we left this as it is, the red pod could still be scheduled on the green node and the red pod could still be scheduled on the grey node. Yet, we have prevented grey pods from being scheduled on our coloured-nodes.
            * Apply node affinity to each coloured-pod, such that the green pod will only be scheduled on the green node and likeswise for the red pod-node and blue pod-node. 
                * Now the coloured-pods will only be scheduled on the node with the matching colour.

<br>

### Multi-Container Pods
* There are three patterns of multi-container pods
    * Sidecar
        * example: loging agent as a second container alongside your application to collect logs and send them to central logging server.
        * here there is a natural 1:1 relationship between an app and its logging agent and so it makes sense for them to share the same lifecycle and scale together. It also makes container communication easier. 
    * Adapter
        * example: a microservices architecture sends logs to a cental logging server but the individual services generate logs in different formats; an adapter pattern is used such that a second container converts the logs to a homogenous format before sending them to the central logging server.
    * Ambassador
        * example: let's say your app speaks to different databases at different stages in its development - you might use a different MySQL DB for dev, QA and prod. You would use a second container here to externalise this DB connection logic from the application and serve as a proxy to the DB so that the app could simply reference local host.

<br>

### Observability
* Pod Status:
    * ``` Pending ``` - waiting to be scheduled on a node
    * ``` ContainerCreating ``` - images pulled and container starts
    * ``` Running ``` - pod is in running state until the container has completed its job or terminates
* Pod Conditions (true/false):
    * ``` PodScheduled ```
    * ``` Initialised ```
    * ``` ContainerReady ```
    * ``` Ready ```
* A ***Readiness Probe*** is a test to check whether or not an application is fully up and running inside a pod.
    * This is required as K8s defines the pod as ``` Running ``` if the container has been created - however a large application may still be starting up when a user sends traffic.
    * You can add a ReadinessProbe to a Pod's spec in the definition YAML; K8s will use this to determine when the Pod is actually ``` Running ```.
    * There are three types of readiness probes:
        * HTTP GET - send a request to an application endpoint
        * TCP socket - send a request to an application port
        * exec - execute a command 
    * It is possible to configure extra properties too:
        * probe failure threshold
        * probe frequency
        * minimum time before first probe
* A ***Liveness Probe*** is a test to check whether or not the application running in a container is actually healthy.
    * A container may be shown to ``` Running ``` but the application inside might be in a crash loop. A liveness probe will destroy and recreate the container if the probe test fails.
    * As above, you can perform a probe using a HTTP endpoint, check if a TCP socket is listening or execute a command.
* ***Logging***
    * Use ``` kubectl logs <pod-name> ``` to get the logs for a pod.
    * If more than one container exists in the pod then you must also specify the container name that you wish to view
        * ``` kubectl logs <pod-name> <container-name> ```
* ***Monitoring & Debugging***
    * There is a single *Metric Server* per K8s cluster, which aggregates logs from each of the nodes and stores them in-memory (ephemeral, not stored to disk). 
    * The kubelet on each node gets logs from each pod and exposes them for the metric server.
        * use ``` kubectl top node/pod ```



<br>

### ```kubectl``` Commands

* Create a pod (imperative)
    * ```kubectl run nginx --image nginx```
* Create a namespace (imperative)
    * ```kubectl create ns dev```
* Generate the YAML for a pod without creating it
    * ``` kubectl run redis --image=redis123 --dry-run=client -o yaml > pod.yaml ```
* Get the previous logs for a pod
    * ``` kubectl logs <pod-name> ```
* Get the stream of logs for a pod
    * ``` kubectl logs -f <pod-name> ```
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
* Explain a K8s Resource (without using the docs)
    * ```kubectl explain <k8s-resource> --recursive | less```
* Encode data to base64
    * ```echo -n 'data' | base64```
* Decode base64 to data
    * ```echo -n 'data' | base64 --decode```
* View a Secret (data hidden)
    * ```kubectl describe <secret-name>```
* View a Secret (data exposed)
    * ```kubectl get secret/<secret-name> -o yaml```
* Discover all the shorthand in ```kubectl```
    * ```kubectl describe```
* Create a Service Account imperatively
    * ```kubectl create serviceaccount <service-account-name>```
* Taint a node
    * ``` kubectl taint nodes <node-name> key=value:taint-effect ```
        e.g ``` kubectl taint nodes node123 app=blue:NoSchedule ```
* Find a Taint
    * ``` kubectl describe node <node-name> | grep taint ```
* Untaint a node (the minus removes the taint)
    * ``` kubectl taint node <node-name> key=value:taint-effect- ```
* Label a node
    * ``` kubectl label node <node-name> key=value ```
* Find the number of labels on a node
    * ``` kubectl get nodes <node-name> --show-labels ```
* View cluster performance
    * ``` kubectl top node ```
    * ``` kubectl top pod ```

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
        securityContext: #optional, pod-lvl security context
          runAsUser: 1000 #root vs non-root
        serviceAccount: <serivce-account-name> #optional
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
                  secretRef:
                    name: <secret-name>
              securityContext: #optional, container-lvl security context, overrides the pod-lvl security context
                runAsUser: 1000
                capabilities: 
                    add: ["MAC_ADMIN"]
              readinessProbe: #optional
                httpGet:
                  path: /api/ready
                  port: 8080
              livenessProbe: #optional
                httpGet:
                  path: /api/healthy
                  port: 8080
            - name: log-agent #optional, multi-container pod
              image: log-agent
        resources: #optional, resource requests
          requests:
            memory: "1Gi"
            cpu: 1
          limits: 
            memory: "2Gi"
            cpu: 2
        tolerations: #optional
        - key: "app"
          operator: "Equal"
          value: "blue"
          effect: "NoSchedule"
        nodeSelector: #optional
          size: Large # key-value label
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

 * Secret: <br>
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: app-secret
    data:
      DB_USER: cm9vdA==
      DB_PASS: c21pZmZ5
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

