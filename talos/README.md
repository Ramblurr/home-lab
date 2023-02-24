
## Order of operations

```
# omitted: create cluster vms

talhelper gensecret > talsecret.sops.yaml
sops -e -i talsecret.sops.yaml

talhelper genconfig

export TALOSCONFIG=~/src/home-lab/talos/clusterconfig/talosconfig
talosctl apply-config --insecure --nodes 192.168.1.104 --file clusterconfig/home-cluster-kmaster0.yaml
talosctl apply-config --insecure --nodes 192.168.1.106 --file clusterconfig/home-cluster-kmaster1.yaml
talosctl apply-config --insecure --nodes 192.168.1.107 --file clusterconfig/home-cluster-kmaster2.yaml
talosctl bootstrap --nodes 192.168.1.104
talosctl kubeconfig -f .

export KUBECONFIG=$(pwd)/kubeconfig
kubectl get no
# wait until all nodes appear

kubectl kustomize --enable-helm ./cni | kubectl apply -f -
kubectl kustomize --enable-helm ./kubelet-csr-approver | kubectl apply -f -
```


Then enable cert rotation
Then regen and reapply talos config

Wait until `kubectl get no` is ready and `cilium status` is green, then proceed with k8s/bootstrap/README
