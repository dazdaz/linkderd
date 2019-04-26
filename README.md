### Links / Background Reading
* https://linkerd.io/
* https://linkerd.io/1/overview/what-is-linkerd/
* https://www.youtube.com/watch?time_continue=1463&v=2trOvMUuLkk Service Mesh evolution - talk at CNCF

### Overview
Deploy linkerd on AKS using supergloo.
SuperGloo, an open-source project to manage and orchestrate service meshes at scale.

### Deploy supergloo
```
curl -sL https://run.solo.io/supergloo/install | sh
export PATH=$PATH:$HOME/.supergloo/bin
supergloo init
supergloo init --dry-run
supergloo install linkerd --name linkerd
kubectl get ns
kubectl get all -n supergloo-system
kubectl get pods -n supergloo-system
kubectl get pod -n linkerd –watch

cat << EOF | kubectl apply -f -
apiVersion: supergloo.solo.io/v1
kind: Install
metadata:
  name: linkerd
spec:
  installationNamespace: linkerd
  mesh:
    linkerdMesh:
      enableAutoInject: true
      enableMtls: true
      linkerdVersion: stable-2.2.1
EOF
```

### Deploy the linkerd CLI
```
curl -sL https://run.linkerd.io/install | sh
export PATH=$PATH:$HOME/.linkerd2/bin
linkerd check --pre
linkerd version
```

### Configure port-forwarding on your local desktop
```
az login
az account set --subscription <subID>
az aks install-cli --install-location c:\apps\kubectl.exe
az aks get-credentials --name k8saks --resource-group k8saks-rg

kubectl get pods -n linkerd
kubectl -n linkerd port-forward linkerd-web-54b47dd7d9-d2crj 8084:8084
```

### Point your browser 
* https://localhost:8084/ Linkerd2 GUI
* http://localhost:8084/grafana/ Grafana GUI

All pods within the linkerd namespace are meshed by default.

### Run a demo app
```
curl -sL https://run.linkerd.io/emojivoto.yml | kubectl apply -f -
kubectl get pods,svc -n emojivoto
```

### Goto linkerd and prove that the app is not meshed
```
http://localhost:9000/overview
```

### Mesh the demo app through injection
```
kubectl get -n emojivoto deploy -o yaml | linkerd inject - | kubectl apply -f -
```

### Change/explore manifest’s meshed settings
```
kubectl edit deployment/voting -n emojivoto
```

### Show app stats using linkerd from the command line
```
linkerd -n emojivoto stat deploy
```

### View services running in realtime
```
linkerd -n emojivoto top deploy
```

### Tap shows the stream of requests across a single pod, deployment, or even everything in the emojivoto namespace.
```
linkerd -n emojivoto tap deploy/web
req id=17:1 proxy=in  src=10.244.1.30:60248 dst=10.244.1.31:80 tls=true :method=GET :authority=web-svc.emojivoto:80 :path=/api/list
req id=17:2 proxy=out src=10.244.1.31:57000 dst=10.244.1.33:8080 tls=true :method=POST :authority=emoji-svc.emojivoto:8080 :path=/emojivoto.v1.EmojiService/ListAll
rsp id=17:2 proxy=out src=10.244.1.31:57000 dst=10.244.1.33:8080 tls=true :status=200 latency=1123µs
end id=17:2 proxy=out src=10.244.1.31:57000 dst=10.244.1.33:8080 tls=true grpc-status=OK duration=42µs response-length=2140B
rsp id=17:1 proxy=in  src=10.244.1.30:60248 dst=10.244.1.31:80 tls=true :status=200 latency=2359µs
```

### Disable/Enable auto-injection
```
kubectl edit install linkerd
```

### Remove linkderd
```
kubectl delete -n default -f https://raw.githubusercontent.com/istio/istio/1.0.6/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl delete ns not-injected
```
