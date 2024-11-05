# Set Up The Jumpbox

In this lab you will set up one of the four machines to be a `jumpbox`. This machine will be used to run commands in this tutorial. While a dedicated machine is being used to ensure consistency, these commands can also be run from just about any machine including your personal workstation running macOS or Linux.

Think of the `jumpbox` as the administration machine that you will use as a home base when setting up your Kubernetes cluster from the ground up. One thing we need to do before we get started is install a few command line utilities and clone the Kubernetes The Hard Way git repository, which contains some additional configuration files that will be used to configure various Kubernetes components throughout this tutorial. 

Log in to the `jumpbox`:

```bash
ssh root@jumpbox
```

All commands will be run as the `root` user. This is being done for the sake of convenience, and will help reduce the number of commands required to set everything up.

### Install Command Line Utilities

Now that you are logged into the `jumpbox` machine as the `root` user, you will install the command line utilities that will be used to preform various tasks throughout the tutorial. 

```bash
apt-get -y install wget curl vim openssl git
```

### Sync GitHub Repository

Now it's time to download a copy of this tutorial which contains the configuration files and templates that will be used build your Kubernetes cluster from the ground up. Clone the Kubernetes The Hard Way git repository using the `git` command:

```bash
git clone --depth 1 \
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```

Change into the `kubernetes-the-hard-way` directory:

```bash
cd kubernetes-the-hard-way
```

This will be the working directory for the rest of the tutorial. If you ever get lost run the `pwd` command to verify you are in the right directory when running commands on the `jumpbox`:

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### Download Binaries

In this section you will download the binaries for the various Kubernetes components. The binaries will be stored in the `downloads` directory on the `jumpbox`, which will reduce the amount of internet bandwidth required to complete this tutorial as we avoid downloading the binaries multiple times for each machine in our Kubernetes cluster.

From the `kubernetes-the-hard-way` directory create a `downloads` directory using the `mkdir` command:

```bash
mkdir downloads
```

The binaries that will be downloaded are listed in the `downloads.txt` file, which you can review using the `cat` command:

```bash
cat downloads.txt
```
Update the downloads.txt file as per [kubernets Download](https://kubernetes.io/releases/download/)

For version 1.31.2 and amd64 architecture, my file name is downloads_kaium.txt as :

```text
https://dl.k8s.io/release/v1.31.2/bin/linux/amd64/kubectl
https://dl.k8s.io/v1.31.2/bin/linux/amd64/kube-apiserver
https://dl.k8s.io/v1.31.2/bin/linux/amd64/kube-controller-manager
https://dl.k8s.io/v1.31.2/bin/linux/amd64/kube-scheduler
https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.31.1/crictl-v1.31.1-linux-amd64.tar.gz
https://github.com/opencontainers/runc/releases/download/v1.2.1/runc.amd64
https://github.com/containernetworking/plugins/releases/download/v1.6.0/cni-plugins-linux-amd64-v1.6.0.tgz
https://github.com/containerd/containerd/releases/download/v1.7.23/containerd-1.7.23-linux-amd64.tar.gz
https://dl.k8s.io/v1.31.2/bin/linux/amd64/kube-proxy
https://dl.k8s.io/v1.31.2/bin/linux/amd64/kubelet
https://github.com/etcd-io/etcd/releases/download/v3.5.16/etcd-v3.5.16-linux-amd64.tar.gz
```
How will find the link:
- kubectl : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kubectl
- kube-apiserver : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kube-apiserver
- kube-controller-manager : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kube-controller-manager
- kube-scheduler : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kube-scheduler
- crictl-v1.31.1 : please visit [[kubernetes-sigs cri-tools]((https://github.com/kubernetes-sigs/cri-tools/releases))](https://github.com/kubernetes-sigs/cri-tools/releases) and click the latest tag. Based on your Operating System,Architecture,please choose your binary link under Assets, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: crictl-v1.31.1-linux-amd64.tar.gz
- runc v1.2.1 : please visit [github for runc](https://github.com/opencontainers/runc/releases/) and click the latest tag. Based on your Operating System,Architecture,please choose your binary link under Assets, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: runc.amd64
- cni-plugins-linux : please visit [github for container networking]([https://github.com/opencontainers/runc/releases/](https://github.com/containernetworking/plugins/releases/)) and click the latest tag. Based on your Operating System,Architecture,please choose your binary link under Assets, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: cni-plugins-linux-amd64-v1.6.0.tgz
- containerd : please visit [github for containerd]([[https://github.com/opencontainers/runc/releases/](https://github.com/containernetworking/plugins/releases/)](https://github.com/containerd/containerd/releases/)) and click the last tag. Based on your Operating System,Architecture,please choose your binary link under Assets, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: containerd-1.7.23-linux-amd64.tar.gz

- kube-proxy : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kube-proxy
- kubelet : please visit [Download Kubernetes](https://kubernetes.io/releases/download/#binaries) . Based on your Operating System,Architecture, Download Binary, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: kubelet
- etcd : please visit [github for etcd](https://github.com/etcd-io/etcd/releases/) and click the lastest tag. Based on your Operating System,Architecture,please choose your binary link under Assets, copy the link. For my case, Operating System: linux, Architecture: amd64, Download Binary: etcd-v3.5.16-linux-amd64.tar.gz

Download the binaries listed in the `downloads_kaium.txt` file using the `wget` command:

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads_kaium.txt
```

Depending on your internet connection speed it may take a while to download the `584` megabytes of binaries, and once the download is complete, you can list them using the `ls` command:

```bash
ls -loh downloads
```

```text
total 584M
-rw-r--r-- 1 root  41M May  9 13:35 cni-plugins-linux-arm64-v1.3.0.tgz
-rw-r--r-- 1 root  34M Oct 26 15:21 containerd-1.7.8-linux-arm64.tar.gz
-rw-r--r-- 1 root  22M Aug 14 00:19 crictl-v1.28.0-linux-arm.tar.gz
-rw-r--r-- 1 root  15M Jul 11 02:30 etcd-v3.4.27-linux-arm64.tar.gz
-rw-r--r-- 1 root 111M Oct 18 07:34 kube-apiserver
-rw-r--r-- 1 root 107M Oct 18 07:34 kube-controller-manager
-rw-r--r-- 1 root  51M Oct 18 07:34 kube-proxy
-rw-r--r-- 1 root  52M Oct 18 07:34 kube-scheduler
-rw-r--r-- 1 root  46M Oct 18 07:34 kubectl
-rw-r--r-- 1 root 101M Oct 18 07:34 kubelet
-rw-r--r-- 1 root 9.6M Aug 10 18:57 runc.arm64
```

### Install kubectl

In this section you will install the `kubectl`, the official Kubernetes client command line tool, on the `jumpbox` machine. `kubectl will be used to interact with the Kubernetes control once your cluster is provisioned later in this tutorial.

Use the `chmod` command to make the `kubectl` binary executable and move it to the `/usr/local/bin/` directory:

```bash
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

At this point `kubectl` is installed and can be verified by running the `kubectl` command:

```bash
kubectl version --client
```

```text
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

At this point the `jumpbox` has been set up with all the command line tools and utilities necessary to complete the labs in this tutorial.

Next: [Provisioning Compute Resources](03-compute-resources.md)