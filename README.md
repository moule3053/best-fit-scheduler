# best-fit-scheduler
Best-fit bin packing scheduler configuration for Kubernetes using the "MostRequestedPriority" priority function of kube-scheduler

## Deployment

To use this scheduler

1. Deploy the `scheduler-config` ConfigMap which specifies the policies to configure kube-scheduler with

`kubectl create -f manifests/scheduler-policy-config.yaml`

2. Deploy another instance of kube-scheduler named `best-fit-policy` which uses the above policy

`kubectl create -f manifests/best-fit-scheduler.yaml`

3. To use this scheduler (and not the default kube-scheduler) specify `spec.schedulerName` to `best-fit-scheduler`. An example deployment is given

`kubectl create -f examples/nginx-deployment.yaml`

## Tests

### Default scheduler

After deploying nginx with default scheduler

`kubectl get po -owide` shows

```
NAME                               READY   STATUS    RESTARTS   AGE   IP          NODE                                     NOMINATED NODE
nginx-deployment-884c7fc54-cxdqc   1/1     Running   0          25s   10.24.2.6   gke-ca-test-default-pool-27725aea-p09h   <none>
```
Now scale the deployment by increasing the number of replicas to 3

`kubect scale deploy nginx-deployment --relicas=3` and check with `kubectl get po -owide` again

```
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE                                     NOMINATED NODE
nginx-deployment-884c7fc54-cxdqc   1/1     Running   0          59s   10.24.2.6    gke-ca-test-default-pool-27725aea-p09h   <none>
nginx-deployment-884c7fc54-kt9pm   1/1     Running   0          3s    10.24.0.23   gke-ca-test-default-pool-27725aea-kzfg   <none>
nginx-deployment-884c7fc54-tdbpx   1/1     Running   0          3s    10.24.1.13   gke-ca-test-default-pool-27725aea-x03n   <none>
```

Notice that the pods are placed on different nodes. This is because kube-scheduler by default uses the `LeastRequestedPriority` for prioritizing nodes.

### best-fit scheduler

After deploying nginx with the best-fit scheduler

`kubectl get po -owide` shows

```
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE                                     NOMINATED NODE
nginx-deployment-667764b9-dpn6q   1/1     Running   0          22s   10.24.0.20   gke-ca-test-default-pool-27725aea-kzfg   <none>
```

Now scale the deployment by increasing the number of replicas to 3

`kubect scale deploy nginx-deployment --relicas=3` and check with `kubectl get po -owide` again

```
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE                                     NOMINATED NODE
nginx-deployment-667764b9-9b9z8   1/1     Running   0          4s    10.24.0.22   gke-ca-test-default-pool-27725aea-kzfg   <none>
nginx-deployment-667764b9-cpv9p   1/1     Running   0          4s    10.24.0.21   gke-ca-test-default-pool-27725aea-kzfg   <none>
nginx-deployment-667764b9-dpn6q   1/1     Running   0          59s   10.24.0.20   gke-ca-test-default-pool-27725aea-kzfg   <none>
```

Notice that all three replicas are places on the same node.

