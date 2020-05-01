# Pre Requisites

## Docker CE
For Fedora 32/31/30: https://computingforgeeks.com/how-to-install-docker-on-fedora/

```
# After following the instructions above
sudo mkdir /sys/fs/cgroup/systemd
sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
sudo dnf install -y grubby
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
sudo reboot
``` 
Good background read regarding cgroup v1/v2:
https://medium.com/nttlabs/cgroup-v2-596d035be4d7


For MacOSX: https://hub.docker.com/editions/community/docker-ce-desktop-mac/
 

## Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod -v +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## cfssl and cfssl-json
https://github.com/cloudflare/cfssl#installation 



## Install golang-14 for linux-amd64
### https://golang.org/doc/install?download=go1.14.2.linux-amd64.tar.gz
```
package=go1.14.2.linux-amd64.tar.gz
curl -O https://dl.google.com/go/$package
sudo tar -C /usr/local -xzf $package && rm -f $package
```

## Install golang-14 for darwin-amd64
### https://golang.org/doc/install?download=go1.14.2.darwin-amd64.pkg
```
package=go1.14.2.darwin-amd64.pkg
curl -O https://dl.google.com/go/$package
sudo open $package
```

## Set up bash profile
```
echo 'export PATH=${PATH}:/usr/local/go/bin' >> ~/.bash_profile
echo 'export GOPATH_K8S=${HOME}/go/src/k8s.io/kubernetes' >> ~/.bash_profile
echo 'export PATH=${GOPATH_K8S}/third_party/etcd:${PATH}' >> ~/.bash_profile

source ~/.bash_profile
```

## Clone kubernetes
```
mkdir -p ${GOPATH_K8S}
git clone https://github.com/kubernetes/kubernetes ${GOPATH_K8S}
cd ${GOPATH_K8S}
mkdir -vp ${GOPATH_K8S}/third_party/etcd
git remote rename origin upstream
```

## Install Etcd
```
${GOPATH_K8S}/hack/install-etcd.sh
```

## Start docker
```
sudo systemctl enable docker && sudo systemctl start docker
```


## Build Kubernetes

```
cd ${GOPATH_K8S}
git checkout v1.12.3
make quick-release
```

## Run a Local Kubernetes Cluster

Start the cluster:

```
${GOPATH_K8S}/hack/local-up-cluster.sh
```

Verify the cluster is running in another ssh window:

```
export KUBERNETES_PROVIDER=local
export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
${GOPATH_K8S}/cluster/kubectl.sh get nodes
```
