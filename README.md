# AKS-Spot-Instances (preview)

### How to create spot instances in AKS

### Make sure you have the latest Az extension AKS-preview updated:

```bash
az extension update --name aks-preview
```
### Enable feature flag:
```bash
az feature register --namespace Microsoft.ContainerService --name SpotPoolPreview
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
