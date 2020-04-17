## install golang-14 for linux-amd64
```
package=go1.14.2.linux-amd64.tar.gz
curl -O https://dl.google.com/go/${package}
sudo tar -C /usr/local -xzf ${package} && rm -f ${package}
```

## set up bash profile
```
echo 'export PATH=${PATH}:/usr/local/go/bin' >> ~/.bash_profile
echo 'export GOPATH_K8S=${HOME}/go/src/k8s.io/kubernetes' >> ~/.bash_profile
echo 'export PATH=${GOPATH_K8S}/third_party/etcd:${PATH}' >> ~/.bash_profile

source ~/.bash_profile
```

## clone kubernetes
```
mkdir -p ${GOPATH_K8S}
git clone https://github.com/kubernetes/kubernetes ${GOPATH_K8S}
cd ${GOPATH_K8S}
mkdir -vp ${GOPATH_K8S}/third_party/etcd
git remote rename origin upstream

# install etcd
./hack/install-etcd.sh

# start docker
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
