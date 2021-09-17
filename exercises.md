# Exercises

<br>

### Find the image used to create a pod
* ``` kubectl describe pod <pod-name> | grep -i image ``` 

<br>

### Find the node which a pod is running on
* ``` kubectl get pods -o wide ``` 

<br>

### Delete a pod
* ``` kubectl delete po/<pod-name> ``` 
    * it will be recreated if deployment still exists...

<br>

### Create a pod imperatively and generate the YAML
* ``` kubectl run redis --image=redis123 -o yaml > pod.yaml ``` 
    * use ``` --dry-run=client ``` to avoid creating the resource

<br>

### Update a replicaset's image - imperative, no access to original YAML
* ``` kubectl edit rs <rs-name> ```
* replace the image field under ```spec/template/spec/containers/image```

<br>

### Find the number of namespaces on a cluster
* ``` kubectl get ns --no-headers | wc -l```
    * ```--no-headers``` leaves out the column headers; ```NAME```, ```STATUS``` and ```AGE``` in this example - makes counting easier/automatic.

<br>

### Create a pod in the ```finance``` ns, with the following spec: ```name=redis```, ```image=redis```
* ``` kubectl run redis -n finance --image=redis --dry-run=client -o yaml > redis.yaml```
* verify the yaml: ```vi redis.yaml```
    * checking that the namespace, name and image fields are correct
* ```kubectl create -f redis.yaml```

<br>

### Which namespace contains the ```blue``` pod?
* ``` kubectl get po --all-namespaces | grep blue```

<br>

### The ```blue``` pod runs in the ```marketing``` ns, alongside a database called ```db-service``` - What DNS name should ```blue``` use to refer to ```db-service```?
* ``` db-service ``` because they are both in the same ns.
    * What DNS name should ```blue``` use to refer to the ```db-service``` that is running in the ```dev``` ns?
        * ```db-service.dev.svc.cluster-local```

<br>

### Using imperative commands, run a pod with the following spec: ```name=redis```, ```image=redis:alpine``` and a label ```tier=db```
* ```kubectl run redis --image=redis:alpine --labels=tier=db```

<br>

### Using imperative commands, expose the redis pod with the following service spec: ```name=redis-service```, ```port=6379```
* ```kubectl expose redis --name redis-service --port 6379 --target-port 6379```

<br>

### Using imperative commands, create a deployment with the following spec: ```name=webapp```, ```image=kodekloud/webapp-color``` and ```replicas=3```
* ```kubectl create deploy/webapp --image=kodekloud/webapp-color```
* ``` kubectl scale deploy/webapp --replicas=3 ```

<br>

### Using imperative commands, create a pod with the following spec: ```name=custom-nginx```, ```image=nginx``` and ```port=8080```
* ```kubectl run custom-nginx --image=nginx --port 8080```

<br>

### Using imperative commands, create a deployment in the ```dev``` ns with the following spec: ```name=redis-deploy```, ```image=redis``` and ```replicas=3```
* ``` kubectl create ns dev ```
* ``` kubectl config set-context $(kubectl config current-context) -n dev ```
* ``` kubectl create deploy/redis-deploy --image=redis --replicas=3 ```

<br>

### Using as few imperative commands as possible, create a pod called ```httpd``` with the ```httpd:apline``` image in the ```default``` ns. Next, create a ```httpd``` svc, of type ```ClusterIP```, with ```target-port=80```
* ``` kubectl run httpd --image=httpd:alpine --port 80 --expose --dry-run=client -o yaml```
    * This lets you check first, confirm it's all good and then re-run
        * ``` kubectl run httpd --image=httpd:alpine --port 80 --expose ```

<br>

### What is the cmd used run the contain in *the* Dockerfile?
* ``` cat Dockerfile ```
    * The answer is the combination of ```ENTRYPOINT``` and ```CMD```

<br>

### What is the environment variable set for the *webapp-color* pod?
* ``` kubectl describe webapp-color | grep env ```

<br>

### Update the environment variable *APP_COLOUR* for the *webapp-color* pod?
* Obtain the yaml from the running pod:
    * ```kubectl get po webapp-color -o yaml > pod.yaml```
* Edit the yaml to update the env var:
    * ```vi pod.yaml```
* Recreate the pod, delete the original if necessary (dependent on deployment, etc)
    * ```kubectl apply -f pod.yaml```

<br>

### Create a config map using the spec provided (imperative)?
* ```kubectl create cm <cm-name> --from-literal=APP_COLOUR=darkblue```

<br>

### Find out how to set a pod env var using a Config Map using the CLI?
* ```kubectl explain pods --recursive | grep envFrom -A3```

<br>

### Create a secret with the following information...
* ```kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2> --from-literal=<key3>=<value3>```
    * type will be ```Opaque```

<br>

### Inject a newly created secret into a running pod
* ``` kubectl get po <pod-name> -o yaml > pod.yaml ```
* ``` vi pod.yaml ```
```yaml
    envFrom:
      - secretRef:
          name: <secret-name>

    delete redundant fields
```
* ``` kubetcl delete po <pod-name> ```
* ``` kubectl apply -f pod.yaml ```
> look up pod yaml: <br>
 ``` kubectl explain pod --recursive```

<br>

### Create a Pod with the following info (info=...., inc Toleration)
* ``` kubectl run bee --image=nginx --restart=Never --dry-run -o yaml > bee.yaml ```
* ``` kubectl explain pod --recursive ```
* ``` vi bee.yaml ``` > add the tolerations section from CLI docs, be careful of the lists in YAML
* ``` kubectl apply -f bee.yaml ```

<br>
 
 ### Remove the taint from the Master node
 * ``` kubectl taint node master node-role.kubernetes.io/master:NoSchedule- ```
    * the minus removes the taint

<br>

### How many labels are there on node01?
* ``` kubectl gets nodes node01 --show-labels ```

<br>

### Add the label 'color=blue' to node01
* ``` kubectl label node node01 color=blue ```

<br>

### Create a deployment with the following info: name = blue, image = nginx, replicas = 6
* ``` kubectl create deploy blue --image=nginx ```
* ``` kubectl scale deploy blue --replicas=6 ```

<br>

### Which node has the pod xyz been scheduled on?
* ``` kubectl get po -o wide ```

<br>

### Set the node affinity to the deployment 'blue' to schedule on node01 only (info provided)
* look up *node affinity* in the k8s docs
* ``` kubectl get deploy blue -o yaml > blue.yaml ```
* add the info to ``` blue.yaml ``` using the format in the k8s docs
* ``` kubectl delete deploy blue ``` 
    * delete to ensure pods restart properly
* ``` kubectl apply -f blue.yaml ```

<br>

### Create a new deployment with the following info, ensure all pods get placed on the master node (node affinity)
* ``` kubectl create deploy red --image=nginx ```
* ``` kubectl scale deploy red --replicas=3 ```
* ``` kubectl get deploy red -o yaml > red.yaml ```
* look up *node affinity* in the k8s docs
* add the ``` <master-label> ``` to red.yaml
    * the ``` operator ``` should be ``` Exists ``` in this case
* ``` kubectl delete deploy red ```
* ``` kubectl apply -f red.yaml ```

<br>

### How many containers are running in the red pod?
* ``` kubectl describe po <pod-name> ```
    * count the containers in the spec section

<br>

### Create a multi-container pod with the following info...
* ``` kubectl run nginx --image nginx --dry-run -o yaml > nginx.yaml ```
    * second container cannot be added imperatively
* ``` vi nginx.yaml ``` and add the second container under the pod spec