# Offline deployment

## Before you begin

* Copy and decompress `releases.tar.gz` into `/tmp/` of all nodes
* Make sure the python version of each node:
  * ctyunos2.0.1: python3, for example, `rm -f /usr/bin/python && ln -s /usr/bin/python3 /usr/bin/python`
  * centos7: python2
* Download and decompress `cce_init_reigstry.tar.gz`, execute `cce_init_registry.sh {registry_endpoint}` to initialize local image registry.

## Deploy

```bash
# 1. download kubespray image
docker pull quay.io/kubespray/kubespray:v2.17.1
# 2. download kubespray source code
git clone -b remotes/origin/release-2.18-ctyunos github.com/jindezgm/kubespray.git
# 3. entry kubespray source code root directory
cd kubespray
# 4. copy sample inventory
cp -rf inventory/sample inventory/cce
# 5. generate host file 
vi inventory/cce/host.yml
# 6. run kubespray container and mount source code into it
docker run --rm -it --mount type=bind,source="$(pwd)",dst=/kubespray --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa quay.io/kubespray/kubespray:v2.17.1 bash
# 7. run ansible
ansible-playbook -i inventory/cce/hosts.yml -e k8s_version=v1.20.2 -e cluster_name=cluster.local -e container_manager=containerd -e kube_proxy_mode=ipvs -e enable_dual_stack_networks=true -e kube_pods_subnet=10.233.64.0/18 -e kube_pods_subnet_ipv6=fd03::/48 -e kube_service_addresses=10.96.0.0/16 -e kube_service_addresses_ipv6=fd05::/112 -e kube_network_plugin=flannel -e flannel_backend_type=vxlan -e flannel_cni_version=v1.0.1 -e flannel_version=v0.16.1 -e ingress_nginx_enabled=true -e ingress_nginx_insecure_port=9080 -e ingress_nginx_secure_port=9443 -e metrics_server_enabled=true -e enable_nodelocaldns=false cluster.yml
```
