# Pre Requisites

## <Your preferred Linux distro here>

```
# Install Docker
https://docs.docker.com/engine/install/

# Optional steps
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

Further info here:
https://docs.docker.com/engine/install/linux-postinstall/
```

## For Fedora 32/31/30 

Get Docker CE 

https://computingforgeeks.com/how-to-install-docker-on-fedora/

```
# After following the instructions above
sudo mkdir /sys/fs/cgroup/systemd
sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
sudo dnf install -y grubby
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
sudo reboot 
``` 

```
# Start docker
sudo systemctl start docker
``` 


Install golang v1.15.6 linux-amd64 

https://golang.org/doc/install?download=go1.15.6.linux-amd64.tar.gz
```
package=go1.15.6.linux-amd64.tar.gz
curl -O https://dl.google.com/go/$package
sudo tar -C /usr/local -xzf $package && rm -f $package
```


## For MacOSX 
Get Docker-CE 

https://hub.docker.com/editions/community/docker-ce-desktop-mac/ 


Install golang 1.15.6 darwin-amd64 

https://golang.org/doc/install?download=go1.15.6.darwin-amd64.pkg
```
package=go1.15.6.darwin-amd64.pkg
curl -O https://dl.google.com/go/$package
sudo open $package
```

Configure the below before continuing  
https://kind.sigs.k8s.io/docs/user/quick-start/#settings-for-docker-desktop


# General setup

You will need to perform the below steps regardless of Linux distribution or MacOSX version. 

## Install Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod -v +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## Install cfssl
https://github.com/cloudflare/cfssl#installation 

```
go get -u github.com/cloudflare/cfssl/cmd/cfssl
export PATH=$PATH:/home/rlourenc/go/bin:

`go env GOPATH`/bin/cfssl} should be available in your $PATH
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

# Ensure etcd binaries are in your $PATH
# Replace etcd version accordingly below
sudo cp -v ${GOPATH_K8S}/third_party/etcd-v3.X.Y-linux-amd64/{etcd,etcdctl} /usr/local/bin/
```


## Build Kubernetes 

> :warning: **zsh on MacOSX**: Ensure you are using the latest version of bash since the build scripts use `mapfile` which isn't supported by zsh. 
 

```
cd ${GOPATH_K8S}

# Obviously you can build any branch. 
# e.g.: git checkout v1.12.3
git checkout master

make quick-release
```


## Note: 
Building and running a Local Kubernetes Cluster from source on MacOS can only be done in a containerized environment such as Kind.
We will see how to make this process work on MacOS in session 2.
At the time of writting, the steps below will not work on MacOS. 

## Run a Local Kubernetes Cluster

Start the cluster:

```
export KUBERNETES_PROVIDER=local
${GOPATH_K8S}/hack/local-up-cluster.sh
...
...
...

Local Kubernetes cluster is running. Press Ctrl-C to shut it down.

Logs:
  /tmp/kube-apiserver.log
  /tmp/kube-controller-manager.log

  /tmp/kube-proxy.log
  /tmp/kube-scheduler.log
  /tmp/kubelet.log

To start using your cluster, you can open up another terminal/tab and run:

  export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
  cluster/kubectl.sh

Alternatively, you can write to the default kubeconfig:

  export KUBERNETES_PROVIDER=local

  cluster/kubectl.sh config set-cluster local --server=https://localhost:6443 --certificate-authority=/var/run/kubernetes/server-ca.crt
  cluster/kubectl.sh config set-credentials myself --client-key=/var/run/kubernetes/client-admin.key --client-certificate=/var/run/kubernetes/client-admin.crt
  cluster/kubectl.sh config set-context local --cluster=local --user=myself
  cluster/kubectl.sh config use-context local
  cluster/kubectl.sh
```

Verify the cluster is running in another ssh window:

```
export KUBERNETES_PROVIDER=local
export KUBECONFIG=/var/run/kubernetes/admin.kubeconfig
${GOPATH_K8S}/cluster/kubectl.sh get nodes

NAME        STATUS   ROLES    AGE    VERSION
127.0.0.1   Ready    <none>   2m3s   v1.21.0-alpha.0.458+0a839c6c3b9489
``` 
