##### VMware NAPP Installation

https://docs.vmware.com/en/VMware-NSX/4.0/nsx-application-platform/GUID-85CD2728-8081-45CE-9A4A-D72F49779D6A.html

I choose the Evaluation t-shirt size installation on top of a Vanilla Kubernetes. The Kuberentes nodes are hosted on top of vSphere 7, and the storage used is vSAN.

The following yaml files must be customized to reflect your environment:
csi-vsan/storage-class.yaml
csi-vsan/vsphere-cloud-controller-manager.yaml
csi-vsan/vsphere-config-secret.yaml

external-dns/external-dns-secret.yaml

metallb/configure.yaml

NAPP uses contour to expose (LoadBalancer svc) its web endpoint. As well, the platform  needs an enpoint for the kafka messaging bus (LoadBalancer svc)
For these requirements, metallb and external-dns (Route53) are used.

Metallb installation:
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
kubectl apply -f metallb/
```

external-dns installation:
```bash
kubectl apply -f external-dns/
```

To provision storage to the Pods, a vSAN CSI backed storage class is used.
vSAN CSI installation:
```bash
kubectl apply -f csi-vsan/
```
