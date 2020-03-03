# AKS-Spot-Instances (preview)

### How to create spot instances in AKS
```bash
Facts before you start:

-Spot pools cannot be primary pool
-Spot needs to be VMSS based
-Spot pools cannot be upgraded
-Spot pools cannot change ScaleSetPriority after creation - recreation is needed
-Spot pools has no SLA and they can be evicted at any time
-Spot Nodes will have label: kubernetes.azure.com/scalesetpriority:spot  and System Pod's having anti-affinity to it.
-Spot Nodes will have taint: kubernetes.azure.com/scalesetpriority=spot:NoSchedule
-Remember to add tolerate to put designated workloads on Spot pool
```

```bash
NB - If you set the max price to be -1:

  -The default price will be up-to on-demand.
  -The instance won't be evicted based on price.
  -The price for the instance will be the current price for Spot or the price for a standard instance, which ever is less, as    long as there is capacity and quota available.
  
  if the value is set to 0.98765 it would be a max price of $0.98765 USD per hour.
```

### Make sure you have the latest Az extension AKS-preview updated:

```bash
az extension update --name aks-preview
```
### Enable feature flag:
```bash
az feature register --namespace Microsoft.ContainerService --name SpotPoolPreview
```

___Output___ 
```bash
{
  "id": "/subscriptions/1111-1111-4afd-111-1111111/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/SpotPoolPreview",
  "name": "Microsoft.ContainerService/SpotPoolPreview",
  "properties": {
    "state": "Registered"
  },
  "type": "Microsoft.Features/providers/features"
}
```

## Create a Spot nodepool
```bash
az aks nodepool add -g gustav-aks --cluster-name gustav-aks-15 -n spotpool1 --priority Spot --spot-max-price -1 --verbose
```
or
```bash
az aks nodepool add -g gustav-aks --cluster-name gustav-aks-15 -n spotpool1 --priority Spot --spot-max-price 1.12345 --verbose
```

```bash
Output:
{
  "agentPoolType": "VirtualMachineScaleSets",
  "availabilityZones": null,
  "count": 3,
  "enableAutoScaling": null,
  "enableNodePublicIp": false,
  "id": "/subscriptions/12a329b9-1659-4afd-83a2-69b873a57e72/resourcegroups/gustav-aks/providers/Microsoft.ContainerService/managedClusters/gustav-aks-15/agentPools/spotpool1",
  "maxCount": null,
  "maxPods": 30,
  "minCount": null,
  "name": "spotpool1",
  "nodeLabels": {
    "scalesetpriority": "spot"
  },
  "nodeTaints": null,
  "orchestratorVersion": "1.15.7",
  "osDiskSizeGb": 100,
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "gustav-aks",
  "scaleSetEvictionPolicy": "Delete",
  "scaleSetPriority": "Spot",
  "spotMaxPrice": -1.0,
  "tags": null,
  "type": "Microsoft.ContainerService/managedClusters/agentPools",
  "vmSize": "Standard_DS2_v2",
  "vnetSubnetId": null
}
```
### View nodes
```Bash
# view nodes
Kubectl get nodes

NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-22572309-vmss000000   Ready    agent   55d   v1.15.7
aks-nodepool1-22572309-vmss000001   Ready    agent   55d   v1.15.7
aks-spotpool1-22572309-vmss000000   Ready    agent   67m   v1.15.7
aks-spotpool1-22572309-vmss000001   Ready    agent   67m   v1.15.7
```
A taint will be added to the Spot nodepools. Ensure you add a tolerate to put worksloads in the designated spotpool. 

```Bash
# View node labels

Kubectl get nodes aks-spotpool1-22572309-vmss000002 --show-labels
NAME                                STATUS   ROLES   AGE   VERSION   LABELS
aks-spotpool1-22572309-vmss000002   Ready    agent   69m   v1.15.7   agentpool=spotpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westeurope,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/cluster=MC_gustav-aks_gustav-aks-15_westeurope,kubernetes.azure.com/role=agent,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-spotpool1-22572309-vmss000002,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,scalesetpriority=spot,storageprofile=managed,storagetier=Premium_LRS
```

## Quick demo with a random container
```Bash
Kubectl apply -f nginx.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    agentpool: spotpool1
```

```Bash
# Container details

Name:         nginx
Namespace:    default
Priority:     0
Node:         aks-spotpool1-22572309-vmss000002/10.240.0.128
Start Time:   Tue, 03 Mar 2020 14:45:49 +0100
Labels:       env=test
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"env":"test"},"name":"nginx","namespace":"default"},"spec":{"contai...
Status:       Running
IP:           10.240.0.129
IPs:          <none>
Containers:
  nginx:
    Container ID:   docker://5aee4da6eaab9f58ad6f5005c6dfe4fbcf6f0f6ec46d51d74b86b2bc408a6cbb
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:380eb808e2a3b0dd954f92c1cae2f845e6558a15037efefcabc5b4e03d666d03
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 03 Mar 2020 14:45:58 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qz8sf (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-qz8sf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qz8sf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  agentpool=spotpool1
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                        Message
  ----    ------     ----  ----                                        -------
  Normal  Scheduled  2m    default-scheduler                           Successfully assigned default/nginx to aks-spotpool1-22572309-vmss000002
  Normal  Pulling    119s  kubelet, aks-spotpool1-22572309-vmss000002  Pulling image "nginx"
  Normal  Pulled     114s  kubelet, aks-spotpool1-22572309-vmss000002  Successfully pulled image "nginx"
  Normal  Created    111s  kubelet, aks-spotpool1-22572309-vmss000002  Created container nginx
  Normal  Started    111s  kubelet, aks-spotpool1-22572309-vmss000002  Started container nginx
  ```
  
