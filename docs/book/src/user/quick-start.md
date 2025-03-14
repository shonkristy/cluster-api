# Quick Start

In this tutorial we'll cover the basics of how to use Cluster API to create one or more Kubernetes clusters.

<aside class="note warning">

<h1>Warning</h1>

If using a [provider] that does not support v1beta1 or v1alpha4 yet, please follow the [release 0.3](https://release-0-3.cluster-api.sigs.k8s.io/user/quick-start.html) or [release 0.4](https://release-0-4.cluster-api.sigs.k8s.io/user/quick-start.html) quickstart instructions instead.

</aside>

## Installation

### Common Prerequisites

- Install and setup [kubectl] in your local environment
- Install [kind] and [Docker]

### Install and/or configure a Kubernetes cluster

Cluster API requires an existing Kubernetes cluster accessible via kubectl. During the installation process the
Kubernetes cluster will be transformed into a [management cluster] by installing the Cluster API [provider components], so it
is recommended to keep it separated from any application workload.

It is a common practice to create a temporary, local bootstrap cluster which is then used to provision
a target [management cluster] on the selected [infrastructure provider].

**Choose one of the options below:**

1. **Existing Management Cluster**

   For production use-cases a "real" Kubernetes cluster should be used with appropriate backup and disaster recovery policies and procedures in place. The Kubernetes cluster must be at least v1.20.0.

   ```bash
   export KUBECONFIG=<...>
   ```
**OR**

2. **Kind**

   <aside class="note warning">

   <h1>Warning</h1>

   [kind] is not designed for production use.

   **Minimum [kind] supported version**: v0.15.0

   **Help with common issues can be found in the [Troubleshooting Guide](./troubleshooting.md).**

   Note for macOS users: you may need to [increase the memory available](https://docs.docker.com/docker-for-mac/#resources) for containers (recommend 6 GB for CAPD).

   Note for Linux users: you may need to [increase `ulimit` and `inotify` when using Docker (CAPD)](./troubleshooting.md#cluster-api-with-docker----too-many-open-files).

   </aside>

   [kind] can be used for creating a local Kubernetes cluster for development environments or for
   the creation of a temporary [bootstrap cluster] used to provision a target [management cluster] on the selected infrastructure provider.

   The installation procedure depends on the version of kind; if you are planning to use the Docker infrastructure provider,
   please follow the additional instructions in the dedicated tab:

   {{#tabs name:"install-kind" tabs:"Default,Docker"}}
   {{#tab Default}}

   Create the kind cluster:
   ```bash
   kind create cluster
   ```
   Test to ensure the local kind cluster is ready:
   ```bash
   kubectl cluster-info
   ```

   {{#/tab }}
   {{#tab Docker}}

   Run the following command to create a kind config file for allowing the Docker provider to access Docker on the host:

   ```bash
   cat > kind-cluster-with-extramounts.yaml <<EOF
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
     extraMounts:
       - hostPath: /var/run/docker.sock
         containerPath: /var/run/docker.sock
   EOF
   ```

   Then follow the instruction for your kind version using  `kind create cluster --config kind-cluster-with-extramounts.yaml`
   to create the management cluster using the above file.

   {{#/tab }}
   {{#/tabs }}

### Install clusterctl
The clusterctl CLI tool handles the lifecycle of a Cluster API management cluster.

{{#tabs name:"install-clusterctl" tabs:"linux,macOS,homebrew,Windows"}}
{{#tab linux}}

#### Install clusterctl binary with curl on linux
Download the latest release; on linux, type:
```bash
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"clusterctl-linux-amd64" version:"1.2.x"}} -o clusterctl
```
Make the clusterctl binary executable.
```bash
chmod +x ./clusterctl
```
Move the binary in to your PATH.
```bash
sudo mv ./clusterctl /usr/local/bin/clusterctl
```
Test to ensure the version you installed is up-to-date:
```bash
clusterctl version
```

{{#/tab }}
{{#tab macOS}}

#### Install clusterctl binary with curl on macOS
Download the latest release; on macOS, type:
```bash
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"clusterctl-darwin-amd64" version:"1.2.x"}} -o clusterctl
```

Or if your Mac has an M1 CPU ("Apple Silicon"):
```bash
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"clusterctl-darwin-arm64" version:"1.2.x"}} -o clusterctl
```

Make the clusterctl binary executable.
```bash
chmod +x ./clusterctl
```
Move the binary in to your PATH.
```bash
sudo mv ./clusterctl /usr/local/bin/clusterctl
```
Test to ensure the version you installed is up-to-date:
```bash
clusterctl version
```
{{#/tab }}
{{#tab homebrew}}

#### Install clusterctl with homebrew on macOS and linux

Install the latest release using homebrew:

```bash
brew install clusterctl
```

Test to ensure the version you installed is up-to-date:
```bash
clusterctl version
```

{{#/tab }}
{{#tab windows}}

#### Install clusterctl binary with curl on Windows using PowerShell
Go to the working directory where you want clusterctl downloaded.

Download the latest release; on Windows, type:
```powershell
curl.exe -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"clusterctl-windows-amd64.exe" version:"1.2.x"}} -o clusterctl.exe
```
Append or prepend the path of that directory to the `PATH` environment variable.

Test to ensure the version you installed is up-to-date:
```powershell
clusterctl.exe version
```

{{#/tab }}
{{#/tabs }}

### Initialize the management cluster

Now that we've got clusterctl installed and all the prerequisites in place, let's transform the Kubernetes cluster
into a management cluster by using `clusterctl init`.

The command accepts as input a list of providers to install; when executed for the first time, `clusterctl init`
automatically adds to the list the `cluster-api` core provider, and if unspecified, it also adds the `kubeadm` bootstrap
and `kubeadm` control-plane providers.

#### Enabling Feature Gates

Feature gates can be enabled by exporting environment variables before executing `clusterctl init`.
For example, the `ClusterTopology` feature, which is required to enable support for managed topologies and ClusterClass,
can be enabled via:
```bash
export CLUSTER_TOPOLOGY=true
```
Additional documentation about experimental features can be found in [Experimental Features].

#### Initialization for common providers

Depending on the infrastructure provider you are planning to use, some additional prerequisites should be satisfied
before getting started with Cluster API. See below for the expected settings for common providers.

{{#tabs name:"tab-installation-infrastructure" tabs:"AWS,Azure,CloudStack,DigitalOcean,Docker,Equinix Metal,GCP,Hetzner,IBM Cloud,Kubevirt,Metal3,Nutanix,OCI,OpenStack,VCD,vcluster,Virtink,vSphere"}}
{{#tab AWS}}

Download the latest binary of `clusterawsadm` from the [AWS provider releases].
{{#tabs name:"install-clusterawsadm" tabs:"linux,macOS,homebrew"}}
{{#tab linux}}

Download the latest release; on linux, type:
```
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-aws" asset:"clusterawsadm-linux-amd64" version:"1.x"}} -o clusterawsadm
```

Make it executable
```
chmod +x clusterawsadm
```

Move the binary to a directory present in your PATH
```
sudo mv clusterawsadm /usr/local/bin
```

Check version to confirm installation
```
clusterawsadm version
```
{{#/tab }}
{{#tab macOS}}

Download the latest release; on macOs, type:
```
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-aws" asset:"clusterawsadm-darwin-amd64" version:"1.x"}} -o clusterawsadm
```

Or if your Mac has an M1 CPU (”Apple Silicon”):
```
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-aws" asset:"clusterawsadm-darwin-arm64" version:"1.x"}} -o clusterawsadm
```

Make it executable
```
chmod +x clusterawsadm
```

Move the binary to a directory present in your PATH
```
sudo mv clusterawsadm /usr/local/bin
```

Check version to confirm installation
```
clusterawsadm version
```
{{#/tab }}
{{#tab homebrew}}

Install the latest release using homebrew:
```
brew install clusterawsadm
```

Check version to confirm installation
```
clusterawsadm version
```

{{#/tab }}
{{#/tabs }}

The [clusterawsadm] command line utility assists with identity and access management (IAM) for [Cluster API Provider AWS][capa].

```bash
export AWS_REGION=us-east-1 # This is used to help encode your environment variables
export AWS_ACCESS_KEY_ID=<your-access-key>
export AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
export AWS_SESSION_TOKEN=<session-token> # If you are using Multi-Factor Auth.

# The clusterawsadm utility takes the credentials that you set as environment
# variables and uses them to create a CloudFormation stack in your AWS account
# with the correct IAM resources.
clusterawsadm bootstrap iam create-cloudformation-stack

# Create the base64 encoded credentials using clusterawsadm.
# This command uses your environment variables and encodes
# them in a value to be stored in a Kubernetes Secret.
export AWS_B64ENCODED_CREDENTIALS=$(clusterawsadm bootstrap credentials encode-as-profile)

# Finally, initialize the management cluster
clusterctl init --infrastructure aws
```

See the [AWS provider prerequisites] document for more details.

{{#/tab }}
{{#tab Azure}}

For more information about authorization, AAD, or requirements for Azure, visit the [Azure provider prerequisites] document.

```bash
export AZURE_SUBSCRIPTION_ID="<SubscriptionId>"

# Create an Azure Service Principal and paste the output here
export AZURE_TENANT_ID="<Tenant>"
export AZURE_CLIENT_ID="<AppId>"
export AZURE_CLIENT_SECRET="<Password>"

# Base64 encode the variables
export AZURE_SUBSCRIPTION_ID_B64="$(echo -n "$AZURE_SUBSCRIPTION_ID" | base64 | tr -d '\n')"
export AZURE_TENANT_ID_B64="$(echo -n "$AZURE_TENANT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_ID_B64="$(echo -n "$AZURE_CLIENT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_SECRET_B64="$(echo -n "$AZURE_CLIENT_SECRET" | base64 | tr -d '\n')"

# Settings needed for AzureClusterIdentity used by the AzureCluster
export AZURE_CLUSTER_IDENTITY_SECRET_NAME="cluster-identity-secret"
export CLUSTER_IDENTITY_NAME="cluster-identity"
export AZURE_CLUSTER_IDENTITY_SECRET_NAMESPACE="default"

# Create a secret to include the password of the Service Principal identity created in Azure
# This secret will be referenced by the AzureClusterIdentity used by the AzureCluster
kubectl create secret generic "${AZURE_CLUSTER_IDENTITY_SECRET_NAME}" --from-literal=clientSecret="${AZURE_CLIENT_SECRET}" --namespace "${AZURE_CLUSTER_IDENTITY_SECRET_NAMESPACE}"

# Finally, initialize the management cluster
clusterctl init --infrastructure azure
```

{{#/tab }}
{{#tab CloudStack}}

Create a file named cloud-config in the repo's root directory, substituting in your own environment's values
```bash
[Global]
api-url = <cloudstackApiUrl>
api-key = <cloudstackApiKey>
secret-key = <cloudstackSecretKey>
```

Create the base64 encoded credentials by catting your credentials file.
This command uses your environment variables and encodes
them in a value to be stored in a Kubernetes Secret.

```bash
export CLOUDSTACK_B64ENCODED_SECRET=`cat cloud-config | base64 | tr -d '\n'`
```

Finally, initialize the management cluster
```bash
clusterctl init --infrastructure cloudstack
```

{{#/tab }}
{{#tab DigitalOcean}}

```bash
export DIGITALOCEAN_ACCESS_TOKEN=<your-access-token>
export DO_B64ENCODED_CREDENTIALS="$(echo -n "${DIGITALOCEAN_ACCESS_TOKEN}" | base64 | tr -d '\n')"

# Initialize the management cluster
clusterctl init --infrastructure digitalocean
```

{{#/tab }}
{{#tab Docker}}

<aside class="note warning">

<h1>Warning</h1>

The Docker provider is not designed for production use and is intended for development environments only.

</aside>

The Docker provider requires the `ClusterTopology` feature to deploy ClusterClass-based clusters. We are
only supporting ClusterClass-based cluster-templates in this quickstart as ClusterClass makes it possible to
adapt configuration based on Kubernetes version. This is required to install Kubernetes clusters < v1.24 and
for the upgrade from v1.23 to v1.24 as we have to use different cgroupDrivers depending on Kubernetes version.

```bash
# Enable the experimental Cluster topology feature.
export CLUSTER_TOPOLOGY=true

# Initialize the management cluster
clusterctl init --infrastructure docker
```

{{#/tab }}
{{#tab Equinix Metal}}

In order to initialize the Equinix Metal Provider (formerly Packet) you have to expose the environment
variable `PACKET_API_KEY`. This variable is used to authorize the infrastructure
provider manager against the Equinix Metal API. You can retrieve your token directly
from the [Equinix Metal Console](https://console.equinix.com/).

```bash
export PACKET_API_KEY="34ts3g4s5g45gd45dhdh"

clusterctl init --infrastructure packet
```

{{#/tab }}
{{#tab GCP}}

```bash
# Create the base64 encoded credentials by catting your credentials json.
# This command uses your environment variables and encodes
# them in a value to be stored in a Kubernetes Secret.
export GCP_B64ENCODED_CREDENTIALS=$( cat /path/to/gcp-credentials.json | base64 | tr -d '\n' )

# Finally, initialize the management cluster
clusterctl init --infrastructure gcp
```

{{#/tab }}
{{#tab Hetzner}}

Please visit the [Hetzner project][Hetzner provider].

{{#/tab }}
{{#tab IBM Cloud}}

In order to initialize the IBM Cloud Provider you have to expose the environment
variable `IBMCLOUD_API_KEY`. This variable is used to authorize the infrastructure
provider manager against the IBM Cloud API. To create one from the UI, refer [here](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui#create_user_key).

```bash
export IBMCLOUD_API_KEY=<you_api_key>

# Finally, initialize the management cluster
clusterctl init --infrastructure ibmcloud
```

{{#/tab }}
{{#tab Kubevirt}}

Please visit the [Kubevirt project][Kubevirt provider].

{{#/tab }}
{{#tab Metal3}}

Please visit the [Metal3 project][Metal3 provider].

{{#/tab }}
{{#tab Nutanix}}

Please follow the Cluster API Provider for [Nutanix Getting Started Guide](https://opendocs.nutanix.com/capx/latest/getting_started/)

{{#/tab }}
{{#tab OCI}}

Please follow the Cluster API Provider for [Oracle Cloud Infrastructure (OCI) Getting Started Guide][oci-provider]

{{#/tab }}
{{#tab OpenStack}}

```bash
# Initialize the management cluster
clusterctl init --infrastructure openstack
```

{{#/tab }}
{{#tab VCD}}

Please follow the Cluster API Provider for [Cloud Director Getting Started Guide](https://github.com/vmware/cluster-api-provider-cloud-director/blob/main/README.md)

EXP_CLUSTER_RESOURCE_SET: "true"
```bash
# Initialize the management cluster
clusterctl init --infrastructure vcd
```

{{#/tab }}
{{#tab vcluster}}

```bash
clusterctl init --infrastructure vcluster
```

Please follow the Cluster API Provider for [vcluster Quick Start Guide](https://github.com/loft-sh/cluster-api-provider-vcluster/blob/main/docs/quick-start.md)

{{#/tab }}
{{#tab Virtink}}

```bash
# Initialize the management cluster
clusterctl init --infrastructure virtink
```

{{#/tab }}
{{#tab vSphere}}

```bash
# The username used to access the remote vSphere endpoint
export VSPHERE_USERNAME="vi-admin@vsphere.local"
# The password used to access the remote vSphere endpoint
# You may want to set this in ~/.cluster-api/clusterctl.yaml so your password is not in
# bash history
export VSPHERE_PASSWORD="admin!23"

# Finally, initialize the management cluster
clusterctl init --infrastructure vsphere
```

For more information about prerequisites, credentials management, or permissions for vSphere, see the [vSphere
project][vSphere getting started guide].

{{#/tab }}
{{#/tabs }}

The output of `clusterctl init` is similar to this:

```bash
Fetching providers
Installing cert-manager Version="v1.9.1"
Waiting for cert-manager to be available...
Installing Provider="cluster-api" Version="v1.0.0" TargetNamespace="capi-system"
Installing Provider="bootstrap-kubeadm" Version="v1.0.0" TargetNamespace="capi-kubeadm-bootstrap-system"
Installing Provider="control-plane-kubeadm" Version="v1.0.0" TargetNamespace="capi-kubeadm-control-plane-system"
Installing Provider="infrastructure-docker" Version="v1.0.0" TargetNamespace="capd-system"

Your management cluster has been initialized successfully!

You can now create your first workload cluster by running the following:

  clusterctl generate cluster [name] --kubernetes-version [version] | kubectl apply -f -
```

<aside class="note">

<h1>Alternatives to environment variables</h1>

Throughout this quickstart guide we've given instructions on setting parameters using environment variables. For most
environment variables in the rest of the guide, you can also set them in ~/.cluster-api/clusterctl.yaml

See [`clusterctl init`](../clusterctl/commands/init.md) for more details.

</aside>

### Create your first workload cluster

Once the management cluster is ready, you can create your first workload cluster.

#### Preparing the workload cluster configuration

The `clusterctl generate cluster` command returns a YAML template for creating a [workload cluster].

<aside class="note">

<h1> Which provider will be used for my cluster? </h1>

The `clusterctl generate cluster` command uses smart defaults in order to simplify the user experience; for example,
if only the `aws` infrastructure provider is deployed, it detects and uses that when creating the cluster.

</aside>

<aside class="note">

<h1> What topology will be used for my cluster? </h1>

The `clusterctl generate cluster` command by default uses cluster templates which are provided by the infrastructure
providers. See the provider's documentation for more information.

See the `clusterctl generate cluster` [command][clusterctl generate cluster] documentation for
details about how to use alternative sources. for cluster templates.

</aside>

#### Required configuration for common providers

Depending on the infrastructure provider you are planning to use, some additional prerequisites should be satisfied
before configuring a cluster with Cluster API. Instructions are provided for common providers below.

Otherwise, you can look at the `clusterctl generate cluster` [command][clusterctl generate cluster] documentation for details about how to
discover the list of variables required by a cluster templates.

{{#tabs name:"tab-configuration-infrastructure" tabs:"AWS,Azure,CloudStack,DigitalOcean,Docker,Equinix Metal,GCP,IBM Cloud,Kubevirt,Metal3,Nutanix,OpenStack,VCD,vcluster,Virtink,vSphere"}}
{{#tab AWS}}

```bash
export AWS_REGION=us-east-1
export AWS_SSH_KEY_NAME=default
# Select instance types
export AWS_CONTROL_PLANE_MACHINE_TYPE=t3.large
export AWS_NODE_MACHINE_TYPE=t3.large
```

See the [AWS provider prerequisites] document for more details.

{{#/tab }}
{{#tab Azure}}

<aside class="note warning">

<h1>Warning</h1>

Make sure you choose a VM size which is available in the desired location for your subscription. To see available SKUs, use `az vm list-skus -l <your_location> -r virtualMachines -o table`

</aside>

```bash
# Name of the Azure datacenter location. Change this value to your desired location.
export AZURE_LOCATION="centralus"

# Select VM types.
export AZURE_CONTROL_PLANE_MACHINE_TYPE="Standard_D2s_v3"
export AZURE_NODE_MACHINE_TYPE="Standard_D2s_v3"

# [Optional] Select resource group. The default value is ${CLUSTER_NAME}.
export AZURE_RESOURCE_GROUP="<ResourceGroupName>"
```

{{#/tab }}
{{#tab CloudStack}}

A ClusterAPI compatible image must be available in your Cloudstack installation. For instructions on how to build a compatible image
see [image-builder (Cloudstack)](https://image-builder.sigs.k8s.io/capi/providers/cloudstack.html)

Prebuilt images can be found [here](http://packages.shapeblue.com/cluster-api-provider-cloudstack/images/)

To see all required Cloudstack environment variables execute:
```bash
clusterctl generate cluster --infrastructure cloudstack --list-variables capi-quickstart
```

Apart from the script, the following Cloudstack environment variables are required.
```bash
# Set this to the name of the zone in which to deploy the cluster
export CLOUDSTACK_ZONE_NAME=<zone name>
# The name of the network on which the VMs will reside
export CLOUDSTACK_NETWORK_NAME=<network name>
# The endpoint of the workload cluster
export CLUSTER_ENDPOINT_IP=<cluster endpoint address>
export CLUSTER_ENDPOINT_PORT=<cluster endpoint port>
# The service offering of the control plane nodes
export CLOUDSTACK_CONTROL_PLANE_MACHINE_OFFERING=<control plane service offering name>
# The service offering of the worker nodes
export CLOUDSTACK_WORKER_MACHINE_OFFERING=<worker node service offering name>
# The capi compatible template to use
export CLOUDSTACK_TEMPLATE_NAME=<template name>
# The ssh key to use to log into the nodes
export CLOUDSTACK_SSH_KEY_NAME=<ssh key name>

```

A full configuration reference can be found in [configuration.md](https://github.com/kubernetes-sigs/cluster-api-provider-cloudstack/blob/master/docs/book/src/clustercloudstack/configuration.md).

{{#/tab }}
{{#tab DigitalOcean}}

A ClusterAPI compatible image must be available in your DigitalOcean account. For instructions on how to build a compatible image
see [image-builder](https://image-builder.sigs.k8s.io/capi/capi.html).

```bash
export DO_REGION=nyc1
export DO_SSH_KEY_FINGERPRINT=<your-ssh-key-fingerprint>
export DO_CONTROL_PLANE_MACHINE_TYPE=s-2vcpu-2gb
export DO_CONTROL_PLANE_MACHINE_IMAGE=<your-capi-image-id>
export DO_NODE_MACHINE_TYPE=s-2vcpu-2gb
export DO_NODE_MACHINE_IMAGE==<your-capi-image-id>
```

{{#/tab }}
{{#tab Docker}}

<aside class="note warning">

<h1>Warning</h1>

The Docker provider is not designed for production use and is intended for development environments only.

</aside>

The Docker provider does not require additional configurations for cluster templates.

However, if you require special network settings you can set the following environment variables:

```bash
# The list of service CIDR, default ["10.128.0.0/12"]
export SERVICE_CIDR=["10.96.0.0/12"]

# The list of pod CIDR, default ["192.168.0.0/16"]
export POD_CIDR=["192.168.0.0/16"]

# The service domain, default "cluster.local"
export SERVICE_DOMAIN="k8s.test"
```

It is also possible but **not recommended** to disable the per-default enabled [Pod Security Standard](../security/pod-security-standards.md):
```bash
export ENABLE_POD_SECURITY_STANDARD="false"
```

{{#/tab }}
{{#tab Equinix Metal}}

There are several required variables you need to set to create a cluster. There
are also a few optional tunables if you'd like to change the OS or CIDRs used.

```bash
# Required (made up examples shown)
# The project where your cluster will be placed to.
# You have to get one from the Equinix Metal Console if you don't have one already.
export PROJECT_ID="2b59569f-10d1-49a6-a000-c2fb95a959a1"
# The facility where you want your cluster to be provisioned
export FACILITY="da11"
# What plan to use for your control plane nodes
export CONTROLPLANE_NODE_TYPE="m3.small.x86"
# What plan to use for your worker nodes
export WORKER_NODE_TYPE="m3.small.x86"
# The ssh key you would like to have access to the nodes
export SSH_KEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDvMgVEubPLztrvVKgNPnRe9sZSjAqaYj9nmCkgr4PdK username@computer"
export CLUSTER_NAME="my-cluster"

# Optional (defaults shown)
export NODE_OS="ubuntu_18_04"
export POD_CIDR="192.168.0.0/16"
export SERVICE_CIDR="172.26.0.0/16"
# Only relevant if using the kube-vip flavor
export KUBE_VIP_VERSION="v0.5.0"
```

{{#/tab }}
{{#tab GCP}}


```bash
# Name of the GCP datacenter location. Change this value to your desired location
export GCP_REGION="<GCP_REGION>"
export GCP_PROJECT="<GCP_PROJECT>"
# Make sure to use same kubernetes version here as building the GCE image
export KUBERNETES_VERSION=1.23.3
# This is the image you built. See https://github.com/kubernetes-sigs/image-builder
export IMAGE_ID=projects/$GCP_PROJECT/global/images/<built image>
export GCP_CONTROL_PLANE_MACHINE_TYPE=n1-standard-2
export GCP_NODE_MACHINE_TYPE=n1-standard-2
export GCP_NETWORK_NAME=<GCP_NETWORK_NAME or default>
export CLUSTER_NAME="<CLUSTER_NAME>"
```

See the [GCP provider] for more information.

{{#/tab }}
{{#tab IBM Cloud}}

```bash
# Required environment variables for VPC
# VPC region
export IBMVPC_REGION=us-south
# VPC zone within the region
export IBMVPC_ZONE=us-south-1
# ID of the resource group in which the VPC will be created
export IBMVPC_RESOURCEGROUP=<your-resource-group-id>
# Name of the VPC
export IBMVPC_NAME=ibm-vpc-0
export IBMVPC_IMAGE_ID=<you-image-id>
# Profile for the virtual server instances
export IBMVPC_PROFILE=bx2-4x16
export IBMVPC_SSHKEY_ID=<your-sshkey-id>

# Required environment variables for PowerVS
export IBMPOWERVS_SSHKEY_NAME=<your-ssh-key>
# Internal and external IP of the network
export IBMPOWERVS_VIP=<internal-ip>
export IBMPOWERVS_VIP_EXTERNAL=<external-ip>
export IBMPOWERVS_VIP_CIDR=29
export IBMPOWERVS_IMAGE_NAME=<your-capi-image-name>
# ID of the PowerVS service instance
export IBMPOWERVS_SERVICE_INSTANCE_ID=<service-instance-id>
export IBMPOWERVS_NETWORK_NAME=<your-capi-network-name>
```

Please visit the [IBM Cloud provider] for more information.

{{#/tab }}
{{#tab Kubevirt}}

A ClusterAPI compatible image must be available in your Kubevirt image library. For instructions on how to build a compatible image
see [image-builder](https://image-builder.sigs.k8s.io/capi/capi.html).

To see all required Kubevirt environment variables execute:
```bash
clusterctl generate cluster --infrastructure kubevirt --list-variables capi-quickstart
```

{{#/tab }}
{{#tab Metal3}}

**Note**: If you are running CAPM3 release prior to v0.5.0, make sure to export the following
environment variables. However, you don't need them to be exported if you use
CAPM3 release v0.5.0 or higher.

```bash
# The URL of the kernel to deploy.
export DEPLOY_KERNEL_URL="http://172.22.0.1:6180/images/ironic-python-agent.kernel"
# The URL of the ramdisk to deploy.
export DEPLOY_RAMDISK_URL="http://172.22.0.1:6180/images/ironic-python-agent.initramfs"
# The URL of the Ironic endpoint.
export IRONIC_URL="http://172.22.0.1:6385/v1/"
# The URL of the Ironic inspector endpoint.
export IRONIC_INSPECTOR_URL="http://172.22.0.1:5050/v1/"
# Do not use a dedicated CA certificate for Ironic API. Any value provided in this variable disables additional CA certificate validation.
# To provide a CA certificate, leave this variable unset. If unset, then IRONIC_CA_CERT_B64 must be set.
export IRONIC_NO_CA_CERT=true
# Disables basic authentication for Ironic API. Any value provided in this variable disables authentication.
# To enable authentication, leave this variable unset. If unset, then IRONIC_USERNAME and IRONIC_PASSWORD must be set.
export IRONIC_NO_BASIC_AUTH=true
# Disables basic authentication for Ironic inspector API. Any value provided in this variable disables authentication.
# To enable authentication, leave this variable unset. If unset, then IRONIC_INSPECTOR_USERNAME and IRONIC_INSPECTOR_PASSWORD must be set.
export IRONIC_INSPECTOR_NO_BASIC_AUTH=true
```

Please visit the [Metal3 getting started guide] for more details.

{{#/tab }}
{{#tab Nutanix}}

A ClusterAPI compatible image must be available in your Nutanix image library. For instructions on how to build a compatible image
see [image-builder](https://image-builder.sigs.k8s.io/capi/capi.html).

To see all required Nutanix environment variables execute:
```bash
clusterctl generate cluster --infrastructure nutanix --list-variables capi-quickstart
```

{{#/tab }}
{{#tab OpenStack}}

A ClusterAPI compatible image must be available in your OpenStack. For instructions on how to build a compatible image
see [image-builder](https://image-builder.sigs.k8s.io/capi/capi.html).
Depending on your OpenStack and underlying hypervisor the following options might be of interest:
* [image-builder (OpenStack)](https://image-builder.sigs.k8s.io/capi/providers/openstack.html)
* [image-builder (vSphere)](https://image-builder.sigs.k8s.io/capi/providers/vsphere.html)

To see all required OpenStack environment variables execute:
```bash
clusterctl generate cluster --infrastructure openstack --list-variables capi-quickstart
```

The following script can be used to export some of them:
```bash
wget https://raw.githubusercontent.com/kubernetes-sigs/cluster-api-provider-openstack/master/templates/env.rc -O /tmp/env.rc
source /tmp/env.rc <path/to/clouds.yaml> <cloud>
```

Apart from the script, the following OpenStack environment variables are required.
```bash
# The list of nameservers for OpenStack Subnet being created.
# Set this value when you need create a new network/subnet while the access through DNS is required.
export OPENSTACK_DNS_NAMESERVERS=<dns nameserver>
# FailureDomain is the failure domain the machine will be created in.
export OPENSTACK_FAILURE_DOMAIN=<availability zone name>
# The flavor reference for the flavor for your server instance.
export OPENSTACK_CONTROL_PLANE_MACHINE_FLAVOR=<flavor>
# The flavor reference for the flavor for your server instance.
export OPENSTACK_NODE_MACHINE_FLAVOR=<flavor>
# The name of the image to use for your server instance. If the RootVolume is specified, this will be ignored and use rootVolume directly.
export OPENSTACK_IMAGE_NAME=<image name>
# The SSH key pair name
export OPENSTACK_SSH_KEY_NAME=<ssh key pair name>
# The external network
export OPENSTACK_EXTERNAL_NETWORK_ID=<external network ID>
```

A full configuration reference can be found in [configuration.md](https://github.com/kubernetes-sigs/cluster-api-provider-openstack/blob/master/docs/book/src/clusteropenstack/configuration.md).

{{#/tab }}
{{#tab VCD}}

A ClusterAPI compatible image must be available in your VCD catalog. For instructions on how to build and upload a compatible image
see [CAPVCD](https://github.com/vmware/cluster-api-provider-cloud-director)

To see all required VCD environment variables execute:
```bash
clusterctl generate cluster --infrastructure vcd --list-variables capi-quickstart
```


{{#/tab }}
{{#tab vcluster}}

```bash
export CLUSTER_NAME=kind
export CLUSTER_NAMESPACE=vcluster
export KUBERNETES_VERSION=1.23.4
export HELM_VALUES="service:\n  type: NodePort"
```

Please see the [vcluster installation instructions](https://github.com/loft-sh/cluster-api-provider-vcluster#installation-instructions) for more details.

{{#/tab }}
{{#tab Virtink}}

To see all required Virtink environment variables execute:
```bash
clusterctl generate cluster --infrastructure virtink --list-variables capi-quickstart
```

See the [Virtink provider](https://github.com/smartxworks/cluster-api-provider-virtink) document for more details.

{{#/tab }}
{{#tab vSphere}}

It is required to use an official CAPV machine images for your vSphere VM templates. See [uploading CAPV machine images][capv-upload-images] for instructions on how to do this.

```bash
# The vCenter server IP or FQDN
export VSPHERE_SERVER="10.0.0.1"
# The vSphere datacenter to deploy the management cluster on
export VSPHERE_DATACENTER="SDDC-Datacenter"
# The vSphere datastore to deploy the management cluster on
export VSPHERE_DATASTORE="vsanDatastore"
# The VM network to deploy the management cluster on
export VSPHERE_NETWORK="VM Network"
# The vSphere resource pool for your VMs
export VSPHERE_RESOURCE_POOL="*/Resources"
# The VM folder for your VMs. Set to "" to use the root vSphere folder
export VSPHERE_FOLDER="vm"
# The VM template to use for your VMs
export VSPHERE_TEMPLATE="ubuntu-1804-kube-v1.17.3"
# The public ssh authorized key on all machines
export VSPHERE_SSH_AUTHORIZED_KEY="ssh-rsa AAAAB3N..."
# The certificate thumbprint for the vCenter server
export VSPHERE_TLS_THUMBPRINT="97:48:03:8D:78:A9..."
# The storage policy to be used (optional). Set to "" if not required
export VSPHERE_STORAGE_POLICY="policy-one"
# The IP address used for the control plane endpoint
export CONTROL_PLANE_ENDPOINT_IP="1.2.3.4"
```

For more information about prerequisites, credentials management, or permissions for vSphere, see the [vSphere getting started guide].

{{#/tab }}
{{#/tabs }}

#### Generating the cluster configuration

For the purpose of this tutorial, we'll name our cluster capi-quickstart.

{{#tabs name:"tab-clusterctl-config-cluster" tabs:"Docker, vcluster, others..."}}
{{#tab Docker}}

<aside class="note warning">

<h1>Warning</h1>

The Docker provider is not designed for production use and is intended for development environments only.

</aside>

```bash
clusterctl generate cluster capi-quickstart --flavor development \
  --kubernetes-version v1.25.0 \
  --control-plane-machine-count=3 \
  --worker-machine-count=3 \
  > capi-quickstart.yaml
```

{{#/tab }}
{{#tab vcluster}}

```bash
export CLUSTER_NAME=kind
export CLUSTER_NAMESPACE=vcluster
export KUBERNETES_VERSION=1.25.0
export HELM_VALUES="service:\n  type: NodePort"

kubectl create namespace ${CLUSTER_NAMESPACE}
clusterctl generate cluster ${CLUSTER_NAME} \
    --infrastructure vcluster \
    --kubernetes-version ${KUBERNETES_VERSION} \
    --target-namespace ${CLUSTER_NAMESPACE} | kubectl apply -f -
```

{{#/tab }}
{{#tab others...}}

```bash
clusterctl generate cluster capi-quickstart \
  --kubernetes-version v1.25.0 \
  --control-plane-machine-count=3 \
  --worker-machine-count=3 \
  > capi-quickstart.yaml
```

{{#/tab }}
{{#/tabs }}

This creates a YAML file named `capi-quickstart.yaml` with a predefined list of Cluster API objects; Cluster, Machines,
Machine Deployments, etc.

The file can be eventually modified using your editor of choice.

See [clusterctl generate cluster] for more details.

#### Apply the workload cluster

When ready, run the following command to apply the cluster manifest.

```bash
kubectl apply -f capi-quickstart.yaml
```

The output is similar to this:

```bash
cluster.cluster.x-k8s.io/capi-quickstart created
dockercluster.infrastructure.cluster.x-k8s.io/capi-quickstart created
kubeadmcontrolplane.controlplane.cluster.x-k8s.io/capi-quickstart-control-plane created
dockermachinetemplate.infrastructure.cluster.x-k8s.io/capi-quickstart-control-plane created
machinedeployment.cluster.x-k8s.io/capi-quickstart-md-0 created
dockermachinetemplate.infrastructure.cluster.x-k8s.io/capi-quickstart-md-0 created
kubeadmconfigtemplate.bootstrap.cluster.x-k8s.io/capi-quickstart-md-0 created
```

#### Accessing the workload cluster

The cluster will now start provisioning. You can check status with:

```bash
kubectl get cluster
```

You can also get an "at glance" view of the cluster and its resources by running:

```bash
clusterctl describe cluster capi-quickstart
```

To verify the first control plane is up:

```bash
kubectl get kubeadmcontrolplane
```

You should see an output is similar to this:

```bash
NAME                    CLUSTER           INITIALIZED   API SERVER AVAILABLE   REPLICAS   READY   UPDATED   UNAVAILABLE   AGE    VERSION
capi-quickstart-g2trk   capi-quickstart   true                                 3                  3         3             4m7s   v1.25.0
```

<aside class="note warning">

<h1> Warning </h1>

The control plane won't be `Ready` until we install a CNI in the next step.

</aside>

After the first control plane node is up and running, we can retrieve the [workload cluster] Kubeconfig:

```bash
clusterctl get kubeconfig capi-quickstart > capi-quickstart.kubeconfig
```

<aside class="note warning">

<h1>Warning</h1>

If you're using Docker Desktop on macOS, or Docker Desktop (Docker Engine works fine) on Linux, you'll need to take a few extra steps to get the kubeconfig for a workload cluster created with the Docker provider. See [Additional Notes for the Docker provider](../clusterctl/developers.md#additional-notes-for-the-docker-provider).

</aside>

### Deploy a CNI solution

Calico is used here as an example.

{{#tabs name:"tab-deploy-cni" tabs:"Azure,vcluster,others..."}}
{{#tab Azure}}

Azure [does not currently support Calico networking](https://docs.projectcalico.org/reference/public-cloud/azure). As a workaround, it is recommended that Azure clusters use the Calico spec below that uses VXLAN.

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  apply -f https://raw.githubusercontent.com/kubernetes-sigs/cluster-api-provider-azure/main/templates/addons/calico.yaml
```

After a short while, our nodes should be running and in `Ready` state,
let's check the status using `kubectl get nodes`:

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig get nodes
```

{{#/tab }}
{{#tab vcluster}}

Calico not required for vcluster.

{{#/tab }}
{{#tab others...}}

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml
```

After a short while, our nodes should be running and in `Ready` state,
let's check the status using `kubectl get nodes`:

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig get nodes
```
```bash
NAME                                          STATUS   ROLES           AGE   VERSION
capi-quickstart-g2trk-9xrjv                   Ready    control-plane   12m   v1.25.0
capi-quickstart-g2trk-bmm9v                   Ready    control-plane   11m   v1.25.0
capi-quickstart-g2trk-hvs9q                   Ready    control-plane   13m   v1.25.0
capi-quickstart-md-0-55x6t-5649968bd7-8tq9v   Ready    <none>          12m   v1.25.0
capi-quickstart-md-0-55x6t-5649968bd7-glnjd   Ready    <none>          12m   v1.25.0
capi-quickstart-md-0-55x6t-5649968bd7-sfzp6   Ready    <none>          12m   v1.25.0
```

{{#/tab }}
{{#/tabs }}

### Clean Up

Delete workload cluster.
```bash
kubectl delete cluster capi-quickstart
```
<aside class="note warning">

IMPORTANT: In order to ensure a proper cleanup of your infrastructure you must always delete the cluster object. Deleting the entire cluster template with `kubectl delete -f capi-quickstart.yaml` might lead to pending resources to be cleaned up manually.
</aside>

Delete management cluster
```bash
kind delete cluster
```

## Next steps

See the [clusterctl] documentation for more detail about clusterctl supported actions.

<!-- links -->
[Experimental Features]: ../tasks/experimental-features/experimental-features.md
[AWS provider prerequisites]: https://cluster-api-aws.sigs.k8s.io/topics/using-clusterawsadm-to-fulfill-prerequisites.html
[AWS provider releases]: https://github.com/kubernetes-sigs/cluster-api-provider-aws/releases
[Azure Provider Prerequisites]: https://capz.sigs.k8s.io/topics/getting-started.html#prerequisites
[bootstrap cluster]: ../reference/glossary.md#bootstrap-cluster
[capa]: https://cluster-api-aws.sigs.k8s.io
[capv-upload-images]: https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/getting_started.md#uploading-the-machine-images
[clusterawsadm]: https://cluster-api-aws.sigs.k8s.io/clusterawsadm/clusterawsadm.html
[clusterctl generate cluster]: ../clusterctl/commands/generate-cluster.md
[clusterctl get kubeconfig]: ../clusterctl/commands/get-kubeconfig.md
[clusterctl]: ../clusterctl/overview.md
[Docker]: https://www.docker.com/
[GCP provider]: https://github.com/kubernetes-sigs/cluster-api-provider-gcp
[Hetzner provider]: https://github.com/syself/cluster-api-provider-hetzner
[IBM Cloud provider]: https://github.com/kubernetes-sigs/cluster-api-provider-ibmcloud
[infrastructure provider]: ../reference/glossary.md#infrastructure-provider
[kind]: https://kind.sigs.k8s.io/
[KubeadmControlPlane]: ../developer/architecture/controllers/control-plane.md
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[management cluster]: ../reference/glossary.md#management-cluster
[Metal3 getting started guide]: https://github.com/metal3-io/cluster-api-provider-metal3/blob/master/docs/getting-started.md
[Metal3 provider]: https://github.com/metal3-io/cluster-api-provider-metal3/
[Kubevirt provider]: https://github.com/kubernetes-sigs/cluster-api-provider-kubevirt/
[oci-provider]: https://oracle.github.io/cluster-api-provider-oci/#getting-started
[Equinix Metal getting started guide]: https://github.com/kubernetes-sigs/cluster-api-provider-packet#using
[provider]:../reference/providers.md
[provider components]: ../reference/glossary.md#provider-components
[vSphere getting started guide]: https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/getting_started.md
[workload cluster]: ../reference/glossary.md#workload-cluster
