
export TALOSCONFIG := ${HOME}/src/home-lab/talos/clusterconfig/talosconfig
export KUBECONFIG := ${HOME}/src/home-lab/talos/kubeconfig



TC := talosctl --talosconfig ${TALOSCONFIG}
KC := kubectl

reset:
	${TC} reset --endpoints 192.168.1.104 --graceful=false -n 192.168.1.104 -n 192.168.1.106 -n 192.168.1.107 --reboot --wait=false
	rm -rf clusterconfig/*.yaml rf clusterconfig/talosconfig kubeconfig talsecret.sops.yaml

gen:
	talhelper gensecret > talsecret.sops.yaml
	sops -e -i talsecret.sops.yaml
	talhelper genconfig

apply-talos-first:
	${TC} apply-config --insecure --nodes 192.168.1.104 --file clusterconfig/home-cluster-kmaster0.yaml
	${TC} apply-config --insecure --nodes 192.168.1.106 --file clusterconfig/home-cluster-kmaster1.yaml
	${TC} apply-config --insecure --nodes 192.168.1.107 --file clusterconfig/home-cluster-kmaster2.yaml

bootstrap:
	${TC} bootstrap --nodes 192.168.1.104

extras:
	${KC} kustomize --enable-helm ./kubelet-csr-approver | kubectl apply -f -
	${KC} kustomize --enable-helm ./cni | kubectl apply -f -

apply-talos:
	${TC} apply-config --nodes 192.168.1.104 --file clusterconfig/home-cluster-kmaster0.yaml
	${TC} apply-config --nodes 192.168.1.106 --file clusterconfig/home-cluster-kmaster1.yaml
	${TC} apply-config --nodes 192.168.1.107 --file clusterconfig/home-cluster-kmaster2.yaml

reboot:
	${TC} reboot --nodes 192.168.1.106 --wait
	${TC} reboot --nodes 192.168.1.107 --wait
	${TC} reboot --nodes 192.168.1.104 --wait

kubeconf:
	${TC} kubeconfig --nodes 192.168.1.104 -f .
