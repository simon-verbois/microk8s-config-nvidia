<p align="center">
  <img src="https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgithub.com%2Fsimon-verbois%2Fmicrok8s-config-nvidia&label=Visitors&countColor=26A65B&style=plastic" alt="Visitor Count" height="28"/>
  <a href="https://github.com/simon-verbois/microk8s-config-nvidia/stargazers"><img src="https://img.shields.io/github/stars/simon-verbois/microk8s-config-nvidia?style=plastic" alt="GitHub Stars" height="28"/></a>
  <a href="https://github.com/simon-verbois/microk8s-config-nvidia/network/members"><img src="https://img.shields.io/github/forks/simon-verbois/microk8s-config-nvidia?style=plastic" alt="GitHub Forks" height="28"/></a>
  <a href="https://github.com/simon-verbois/microk8s-config-nvidia/commits/main"><img src="https://img.shields.io/github/last-commit/simon-verbois/microk8s-config-nvidia?style=plastic" alt="GitHub Last Commit" height="28"/></a>
</p>

# Requirements
- [Nvidia Driver](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/) should be installed on the host
- [Nvidia Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/1.17.8/install-guide.html) should be installed on the host 

<br>

# Note
- This implementation has been tested on RHEL 9
- There is an alias setup on my profile: `alias kubectl='microk8s kubectl'`
- All this implementation has been done with a non root user (in microk8s group)

<br>

# Setup the GPU Operator with time-slicing
```bash
# Enable the module
microk8s enable nvidia

# Check all pods status
kubectl get pods -n gpu-operator-resources -w

# Ensure the initial setup is OK
kubectl describe node | grep nvidia.com/gpu
## You should see a 1 on the right on the nvidia.com/gpu

## Create and apply the configmap
kubectl apply -n gpu-operator-resources -f time-slicing-config-all.yaml

## Patch the live policy
kubectl patch clusterpolicies.nvidia.com/cluster-policy -n gpu-operator-resources --type merge -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config-all", "default": "any"}}}}'
```

<br>

# Check if everything is OK
```bash
# Check event in the NS
kubectl get events -n gpu-operator-resources --sort-by='.lastTimestamp'

# Check all pods status
kubectl get pods -n gpu-operator-resources -w

# Check if the GPU parts are available
kubectl describe node | grep nvidia.com/gpu
## You should see the number of part you add in your time-slicing on the right on the nvidia.com/gpu
```

<br>

# Testing the GPU with time slicing
```bash
# Create the testing pod
kubectl apply -f testing.yaml

# Wait for the pod to complete
kubectl get pods -w

# Check if SMI work correctly
kubectl logs -f nvidia-smi-test

# Cleanup the pod
kubectl delete -f testing.yaml
```

