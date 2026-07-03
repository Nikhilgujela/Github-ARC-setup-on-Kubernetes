# Setup controller in Kubernetes Using Helm Chart 

NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller

# to checl 
$ kubectl get all -n arc-systems
NAME                                         READY   STATUS    RESTARTS   AGE
pod/arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          28s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/arc-gha-rs-controller   1/1     1            1           28s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/arc-gha-rs-controller-7c668d7847   1         1         1       28s

 $ kubectl get pods -n arc-systems
O/P
NAME                                     READY   STATUS    RESTARTS   AGE
arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          42s

# Setup PAT token in Github account & Paste in below config :
setting > developer setting > Personal access token > fine-grained personal access token


# Configuring a runner scale set

INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/<your_enterprise/org/repo>"
GITHUB_PAT="<PAT>"
helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set



helm list -A


kubectl get autoscalingrunnersets -n arc-runners
kubectl get ephemerarunnersets -n arc-runners
kubectl get ephemerarunners -n arc-runners

## after setup runnerset- you can see listener in arc-systems
$ kubectl get pods -n arc-systems 
NAME                                     READY   STATUS    RESTARTS   AGE
arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          22m
arc-runner-set-d7ffb588-listener         1/1     Running   0          92s
root@controlplane:~$ 

kubectl get pods -n arc-runners -w
