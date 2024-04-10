# NeSI CAPI Workload

## Note

You will need access to a CAPI management cluster to submit the following repo's code.

To bootstrap a CAPI management cluster please look at the following repo [NeSI RDC CAPI Management Cluster](https://github.com/nesi/nesi.rdc.kind-bootstrap-capi)

The CAPI images are generated from the following image bulder repo [CAPI Images](https://github.com/lbrick/image-builder/tree/2023-nesi_images)

## Setup

Make a copy of the folder `example` in the `environments` folder and rename it to something that reflects your environment name example `environments/my-test` and fill in the requried parameters within `variables.yml`

``` { .sh }
cp environments/example environments/my-test
```

Inside the `environments/my-test/variables.yml` file is some user configuration required.

``` { .sh }
kubernetes_version: v1.27.6
capi_image_name: rocky-89-kube-v1.27.6

cluster_rdc_project: NeSI_RDC_PROJECT_NAME
cluster_name: CAPI_CLUSTER_NAME
cluster_namespace: default

capi_management_cluster: MANGEMENT_CLUSTER_NAME

openstack_ssh_key: NeSI_RDC_KEYPAIR_NAME

cluster_control_plane_count: 1
control_plane_flavor: balanced1.2cpu4ram

cluster_worker_count: 2
worker_flavour: balanced1.2cpu4ram

cluster_node_cidr: 10.10.0.0/24
cluster_pod_cidr: 192.168.0.0/16
cluster_route_id: 3c0cb930-2bbe-4c9c-ac61-6dbc9410c3e9

clouds_yaml_location: ~/.config/openstack/clouds.yaml
clouds_yaml_cloud: openstack

kube_config_local_location: ~/.kube/config
kube_config_location: "~/.kube"

kube_oidc_auth: false
```

`NeSI_RDC_PROJECT_NAME` needs to be replaced with your NeSI RDC Project name e.g. NeSI-Training-Test

`CAPI_CLUSTER_NAME` the name of your cluster

`MANGEMENT_CLUSTER_NAME` the name of your management cluster that this workload cluster will deploy from

`NeSI_RDC_KEYPAIR_NAME` the name of your keypair that is in the NeSI RDC

There are the following CAPI images available

``` { .sh }
Rocky 9.3

rocky-93-kube-v1.26.12
rocky-93-kube-v1.27.6
rocky-93-kube-v1.28.5


Ubuntu 22.04

ubuntu-2204-kube-v1.26.7
ubuntu-2204-kube-v1.27.6
ubuntu-2204-kube-v1.28.3
```

For workload clusters we recommend Kuberenetes version 1.27+

If changing the `capi_image_name` within `variables.yml` please also ensure the `kubernetes_version` matches the same.

Example would be using the image `rocky-93-kube-v1.27.6` would mean the `kubernetes_version` would be `v1.27.6`

## Run

Running the below command will start the anisble role and deploy a workload cluster

``` { .sh }
ansible-playbook setup-workload.yml
```

To get the kubeconfig from the management cluster for your newly created workload cluster run the following while replacing `CLUSTER_NAME` with the name of your cluster

``` { .sh }
kubectl get secret CLUSTER_NAME-kubeconfig -o jsonpath='{.data.value}' | base64 --decode
```

## Using Cluster autoscaler

If the variable `cluster_max_worker_count` is greater then `cluster_worker_count` then Cluster Autoscaler will be deployed inside the workload cluster. This will not scale the GPU nodes if they are enabled at this stage.

## Using nodes with GPU access

If you have access to GPU flavor's within your RDC project space then you will need to set the variable `enable_gpu_nodes` to `true`

This will deploy a GPU node with the flavor `gpu1.44cpu240ram.a40.1g.48gb` and add it to the workload cluster

The following GPU CAPI images are avaliable:

``` { .sh }
Rocky 8.9

rocky-89-kube-nvidia-v1.27.7

```
