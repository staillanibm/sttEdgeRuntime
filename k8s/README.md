# Deployment of an edge runtime in Kubernetes

The following is to create an edge runtime in Kubernetes.
For OpenShift there are a few specificities and the following procedure won't work out of the box.

## Namespace creation

It's advised to create a dedicated namespace for your runtime.  
Set the K8S_NAMESPACE environment variable to the name of this namespace, then run `kubectl create ns ${K8S_NAMESPACE}`

## Registry credentials

The deployment needs a iwhi-regcred that can be created.  
Set the WHI_CR_SERVER, WHI_CR_USERNAME and WHI_CR_PASSWORD environment variables accordingly before running the following command.  
```
kubectl create secret docker-registry iwhi-regcred \
    --docker-server=${WHI_CR_SERVER} \
    --docker-username=${WHI_CR_USERNAME} \
    --docker-password=${WHI_CR_PASSWORD} \
    --namespace=${K8S_NAMESPACE}
```  

## Edge runtime secret

The pairing URL and credentials are stored in a edge-secret secret.
Set the SAG_IS_CLOUD_REGISTER_URL, SAG_IS_EDGE_CLOUD_ALIAS and SAG_IS_CLOUD_REGISTER_TOKEN environment variable values accordingly before running the following command.
```
kubectl create secret generic edge-secret \
	--from-literal=SAG_IS_CLOUD_REGISTER_URL=${SAG_IS_CLOUD_REGISTER_URL} \
	--from-literal=SAG_IS_EDGE_CLOUD_ALIAS=${SAG_IS_EDGE_CLOUD_ALIAS} \
    --from-literal=SAG_IS_CLOUD_REGISTER_TOKEN=${SAG_IS_CLOUD_REGISTER_TOKEN} \
    --namespace=${K8S_NAMESPACE}
```

Alternatively, you can set these values in the provided secrets.yaml manifest and the run 
```
kubectl apply -f secrets.yaml --namespace ${${K8S_NAMESPACE}}
```    
You can also use the secret manifest to change secret values.  

## Apply the manifests

The deploy.yaml manifest creats a service account and a deployment of one single edge runtime pod. To apply it:
```
kubectl apply -f deploy.yaml --namespace ${K8S_NAMESPACE}
```

## Check the deployment

Use the following command to follow the deployment:
```
kubectl get pods --namespace ${K8S_NAMESPACE}
```

Which should return
```
NAME                            READY   STATUS    RESTARTS   AGE
edge-runtime-xxxxxxxxxx-yyyyy   0/1     Running   0          2m4s
```

A pod will usually take one or two minutes to start. You should see "READY 1/1" when the edge runtime is ready.
```
NAME                            READY   STATUS    RESTARTS   AGE
edge-runtime-xxxxxxxxxx-yyyyy   1/1     Running   0          2m4s
```

To access the container logs, use
```
kubectl logs edge-runtime-xxxxxxxxxx-yyyyy --namespace ${K8S_NAMESPACE}
```
(you need to replace edge-runtime-xxxxxxxxxx-yyyyy with the real name of your pod.)

If the status of you pod is not "Running", check the output of the following command:
```
kubectl describe pod edge-runtime-xxxxxxxxxx-yyyyy --namespace ${K8S_NAMESPACE}
```
(you need to replace edge-runtime-xxxxxxxxxx-yyyyy with the real name of your pod.)