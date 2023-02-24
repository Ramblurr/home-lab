# Bootstrap

## Flux

### Authenticate Flux

[source](https://github.com/onedr0p/flux-cluster-template/tree/164d81a77c69b0c240af4f3b937778ddc22e5661#-authenticate-flux-over-ssh)

1. Generate new SSH key:
      ```sh
      ssh-keygen -t ecdsa -b 521 -C "github-deploy-key" -f ./kubernetes/bootstrap/github-deploy.key -q -P ""
      ```
2. Paste public key in the deploy keys section of your repository settings
3. Create sops secret in `./kubernetes/bootstrap/github-deploy-key.sops.yaml` with the contents of:
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: github-deploy-key
        namespace: flux-system
    stringData:
        # 3a. Contents of github-deploy-key
        identity: |
            -----BEGIN OPENSSH PRIVATE KEY-----
                ...
            -----END OPENSSH PRIVATE KEY-----
        # 3b. Output of curl --silent https://api.github.com/meta | jq --raw-output '"github.com "+.ssh_keys[]'
        known_hosts: |
            github.com ssh-ed25519 ...
            github.com ecdsa-sha2-nistp256 ...
            github.com ssh-rsa ...
    ```
4. Encrypt secret:
    ```sh
    sops --encrypt --in-place ./kubernetes/bootstrap/github-deploy-key.sops.yaml
    ```
5. Apply secret to cluster:
    ```sh
    sops --decrypt ./kubernetes/bootstrap/github-deploy-key.sops.yaml | kubectl apply -f -
    ```
6.  Update `./kubernetes/flux/config/cluster.yaml`:
    ```yaml
    apiVersion: source.toolkit.fluxcd.io/v1beta2
    kind: GitRepository
    metadata:
    name: home-kubernetes
    namespace: flux-system
    spec:
    interval: 10m
    # 6a: Change this to your user and repo names
    url: ssh://git@github.com/$user/$repo
    ref:
        branch: main
    secretRef:
        name: github-deploy-key
    ```
7. Commit and push changes
8. Force flux to reconcile your changes
    ```sh
    task cluster:reconcile
    ```
9. Verify git repository is now using SSH:
    ```sh
    task cluster:gitrepositories
    ```
10. Optionally set your repository to Private in your repository settings.


### Install Flux

```sh
kubectl apply --server-side --kustomize ./k8s/bootstrap
```

### Apply Cluster Configuration

```sh
sops -d k8s/bootstrap/age-key.sops.yaml | kubectl apply -f -
sops -d k8s/bootstrap/github-deploy-key.sops.yaml | kubectl apply -f -
sops -d k8s/flux/vars/global-secrets.yaml | kubectl apply -f -
kubectl apply -f k8s/flux/vars/global-vars.yaml
```

Wait until `kubectl get pods -n flux-system` shows all ready.

### Kick off Flux applying this repository

```sh
kubectl apply --server-side --kustomize ./k8s/flux/config
```
