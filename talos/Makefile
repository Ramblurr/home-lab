.ONESHELL:
.SHELLFLAGS += -e -o pipefail

export TALOSCONFIG := ${HOME}/src/home-lab/talos/clusterconfig/talosconfig
export KUBECONFIG := ${HOME}/src/home-lab/talos/kubeconfig



TC := talosctl --talosconfig ${TALOSCONFIG}
KC := kubectl

reset:
	${TC} reset --endpoints 192.168.1.104 --graceful=false -n 192.168.1.104 -n 192.168.1.106 -n 192.168.1.107 --reboot --wait=false
	rm -rf clusterconfig/*.yaml clusterconfig/talosconfig kubeconfig talsecret.sops.yaml

gen:
	talhelper gensecret > talsecret.sops.yaml
	sops -e -i talsecret.sops.yaml
	talhelper genconfig

apply-talos-first:
	${TC} apply-config --insecure --nodes 192.168.1.104 --file clusterconfig/test-cluster-kmaster0-test.int.socozy.casa.yaml
	${TC} apply-config --insecure --nodes 192.168.1.106 --file clusterconfig/test-cluster-kmaster1-test.int.socozy.casa.yaml
	${TC} apply-config --insecure --nodes 192.168.1.107 --file clusterconfig/test-cluster-kmaster2-test.int.socozy.casa.yaml

bootstrap:
	${TC} bootstrap --nodes 192.168.1.104

extras:
	${KC} kustomize --enable-helm ./cni | kubectl apply -f -
	${KC} kustomize --enable-helm ./kubelet-csr-approver | kubectl apply -f -

apply-talos:
	${TC} apply-config --nodes 192.168.1.104 --file clusterconfig/test-cluster-kmaster0-test.int.socozy.casa.yaml
	${TC} apply-config --nodes 192.168.1.106 --file clusterconfig/test-cluster-kmaster1-test.int.socozy.casa.yaml
	${TC} apply-config --nodes 192.168.1.107 --file clusterconfig/test-cluster-kmaster2-test.int.socozy.casa.yaml

reboot:
	${TC} reboot --nodes 192.168.1.106 --wait
	${TC} reboot --nodes 192.168.1.107 --wait
	${TC} reboot --nodes 192.168.1.104 --wait

kubeconf:
	${TC} kubeconfig --nodes 192.168.1.104 -f .


nuke-and-pave:
	$(MAKE) reset
	$(MAKE) gen
	sleep 120
	$(MAKE) apply-talos-first
	sleep 60
	$(MAKE) bootstrap
	sleep 60
	$(MAKE) kubeconf
	kubectl get no
	talosctl --nodes 192.168.1.104 get members
	$(MAKE) extras
	sleep 60
	talhelper genconfig -c talconfig.yaml-2
	$(MAKE) apply-talos
	kubectl get no
	sleep 60
	kubectl get no
	talosctl --nodes 192.168.1.104 get members
	cilium status
	cilium connectivity test


bootstrap-flux:
	cd ../
	kubectl apply --server-side --kustomize ./k8s/bootstrap
	sops -d k8s/bootstrap/age-key.sops.yaml | kubectl apply -f -
	sops -d k8s/bootstrap/github-deploy-key.sops.yaml | kubectl apply -f -
	sops -d k8s/flux/vars/global-secrets.yaml | kubectl apply -f -
	kubectl apply -f k8s/flux/vars/global-vars.yaml
	kubectl get pods -n flux-system
	sleep 10
	kubectl apply --server-side --kustomize ./k8s/flux/config
	sleep 10
	flux reconcile -n flux-system source git flux-system
	flux reconcile -n flux-system kustomization cluster
