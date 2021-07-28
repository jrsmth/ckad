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