# best-fit-scheduler
Best-fit bin packing scheduler configuration for Kubernetes using the "MostRequestedPriority" priority function of kube-scheduler

To use this scheduler

1. Deploy the `scheduler-config` ConfigMap which specifies the policies to configure kube-scheduler with
`kubectl create -f manifests/scheduler-policy-config.yaml`

2. Deploy another instance of kube-scheduler named `best-fit-policy` which uses the above policy
`kubectl create -f manifests/best-fit-scheduler.yaml`

3. To use this scheduler (and not the default kube-scheduler) specify `spec.schedulerName` to `best-fit-scheduler`. An example deployment is given
`kubectl create -f examples/nginx-deployment.yaml`
