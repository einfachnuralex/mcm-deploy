# Deploy Gardener machine controller manager

Requirements:
- Build a vm image and upload to Openstack (see image/)
- K8s 1.19 cluster with at least one master installed with kubeadm
- CNI installed (calico or whatever)
- Run 'kubeadm token create --print-join-command --ttl 8800' to get a 1y valid join command
- Set fill needed field in values.yaml
  - Put join command to secret userdata command part
- Install helm chart
- Scale machinedeployment as needed

# Build image with packer

sed K8SVERSION to expected version

```
packer build -var "k8sversion=1.19.8" -var 'fipID=network-public-id' -var 'network=network-id' -var 'secgroup=packer-ssh' -var 'visibility=private'
```

# Install MCM with helm chart

```
helm upgrade --install mcm-name ./ -n mcm -f values-custom.yaml
# On clusters with another mcm
helm upgrade --install mcm-name ./ -n mcm -f values-custom.yaml --skip-crds 
```

# Migrate OpenStackMachineClass to MachineClass with version >0.35

There is a automatic migrate from OpenStackMachineClass to MachineClass implemented in >0.35.

For that migration there are two flags in values.yaml:

```yaml
pre_migrate: false # deploys OpenStackMachineClass 
post_migrate: true # deploys MachineClass
```