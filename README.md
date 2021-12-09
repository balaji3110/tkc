# Support Bundle Collection
You can collect a support bundle for a Tanzu Kubernetes cluster using a CLI or using a Docker image (for development purpose only).

## Index
1. [TKC Support Bundler](#tkc-support-bundler)
    * [Download and Install binary](#download-and-install-binary)
    * [Building and Installing the CLI](#building-and-installing-the-cli)
    * [Collecting support bundle using CLI](#collecting-support-bundle-using-cli)
1. [Docker Image](#docker-image)
    * [Building the image](#building-the-image)
    * [Collecting support bundle using docker image](#collecting-support-bundle-using-docker-image)

## TKC Support Bundler
You can use the `tkc-support-bundler` CLI to collect a support bundle from any machine that can connect to:
* The Supervisor Cluster (if you are using NSX-T for the infrastructure network)
* The target Tanzu Kubernetes cluster (if you are using VDS for the infrastructure network)

> Note: For an environment with NSX-T based networking, you will need to have connection to the VMware registry. In case you have an air-gapped environment, make sure to load the image into your private registry which the Supervisor cluster can access.

### Download and Install binary
You can download the latest version of the CLI from the [releases](https://gitlab.eng.vmware.com/guest-clusters/tkg-service-tools/-/releases) page of this repository.
Instructions to run the CLI are in the ["Collecting support bundle using CLI"](#collecting-support-bundle-using-cli) section below.

### Building and Installing the CLI
* To build the CLI run:
```bash
$ make build-cli
```
This command builds the CLI binary in `tkg-service-tools/tkc-support-bundler/bin`.

* To install the CLI run:
```bash
$ make install-cli
```
This command places the `tkc-support-bundler` binary in the directory `/usr/local/bin` in the system PATH, so the CLI is globally available.

To verify installation, run the command `tkc-support-bundler`.

### Collecting support bundle using CLI
To collect the support bundle you need following inputs:
* Absolute path to the Supervisor Cluster kubeconfig on the host executing the CLI (if necessary, copy the file to the host where you are collecting the support bundle)
* Path to an output directory where the final support-bundle is stored
* Name of the Tanzu Kubernetes cluster
* Supervisor Namespace where the Tanzu Kubernetes cluster resides
* Size for the Persistent Volume claim attached to the podVM to store support bundle intermediately
* IP of the VMService based SSH proxy to the Tanzu Kubernetes cluster nodes
* Container image to collect the support bundle. Defaults to `projects.registry.vmware.com/tkg/tkc-support-bundler:v1.3.0`

Run the CLI as:
```bash
$ tkc-support-bundler create \
-k <absolute-path-to-supervisor-cluster-kubeconfig> \
-o <output-dir-on-host> \
-c <tanzu-kubernetes-cluster-name> \
-n <supervisor-namespace-where-TKC-resides> \
-i <container-image> \
-p <pvc-size> \
-j <vm-service-ssh-proxy-ip> \
-t <timeout-for-bundle-collection>
```

> Note: For an air-gapped environment with NSX-T based networking, provide the private registry URL of the image using the `-i` parameter.

The CLI generates the support bundle (TAR) file and saves it to the specified output directory.

## Docker Image

The Docker image is available for development purpose only.

The Docker image is available in the VMware Artifactory at: `projects.registry.vmware.com/tkg/tkc-support-bundler:v1.3.0`

You can also build the Docker image locally and use it for development.

### Building the image
To build the docker image locally run:
```bash
$ make image
```
The image is built on the local host and named `tkc-support-bundler`.
```bash
$ docker images
REPOSITORY                                                                 TAG                 IMAGE ID            CREATED             SIZE
tkc-support-bundler                                                        latest              dbe0bd4d9083        4 days ago          205MB
```
### Collecting support bundle using docker image
To collect the support bundle you will need multiple inputs to the container:
* Mount an output directory on the host, mapped to `/support-bundle` in the container, where the final support-bundle is stored
* Mount the Tanzu Kubernetes Cluster kubeconfig from the host, mapped to `/config/value`
* Mount the Tanzu Kubernetes Cluster ssh key file mapped to `/ssh-key/ssh-privatekey`

You will also need to pass multiple arguments to `crashd` running in the container:
* Tanzu Kubernetes cluster name to collect bundle for, for example `--args cluster=my-tkc`
* Node IPs on the Tanzu Kubernetes cluster, for example `--args nodeips=172.168.10.3 172.168.10.4 172.168.10.5`
* Node hostnames of Tanzu Kubernetes cluster (same sequence as the IPs above), for example `--args nodenames=my-tkc-control-plane my-tkc-worker-01 my-tkc-worker-02`
* IP of the VMService based SSH proxy (if available) to the Tanzu Kubernetes cluster nodes, for example `--args sshproxy=192.168.128.3`

To collect the support bundle run the following command:
```
$ docker run -v <output-dir-on-host>:/support-bundle \
-v <absolute-path-to-tanzu-kubernetes-cluster-kubeconfig>:/config/value \
-v <absolute-path-to-tanzu-kubernetes-cluster-ssh-key>:/ssh-key/ssh-privatekey \
<support-bundle-image> \
--args cluster=<tanzu-kubernetes-cluster-name> \
--args nodeips=<tanzu-kubernetes-cluster-node-ips> \
--args nodenames=<tanzu-kubernetes-cluster-node-hostnames> \
--args sshproxy=<vm-service-ssh-proxy-ip>
```

The CLI generates the support-bundle TAR file and saves it to the directory mounted at `/support-bundle`.
