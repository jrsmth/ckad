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

 