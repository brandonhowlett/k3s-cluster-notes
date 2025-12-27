k3s + Calico (Operator-Based) Installation Guide

Single-node now, multi-node ready

This guide reflects the current, correct installation model using the Tigera Calico operator with no legacy IPPool resources. It is suitable as a repeatable blueprint for bare-metal rebuilds and future multi-node expansion.

1. Install k3s (without Flannel or Traefik)

Calico will fully replace Flannel, and Traefik will be installed separately via Helm.

```bash
curl -sfL https://get.k3s.io
 |
K3S_KUBECONFIG_MODE="644"
INSTALL_K3S_EXEC="
--flannel-backend=none
--cluster-cidr=10.10.250.0/24
--disable-network-policy
--disable=traefik"
sh -
```

Notes

`--cluster-cidr` must match the Calico IP pool CIDR.

NetworkPolicy is disabled here because Calico will manage it.

Traefik is disabled because it will be installed via Helm.

2. Configure kubeconfig for the current user

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
```

Verify access:

```bash
kubectl get nodes
```

3. Install Calico operator CRDs and controller

Install the Calico operator components only (no networking yet):

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/operator-crds.yaml

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/tigera-operator.yaml

```

Wait until the operator is ready:

```bash
kubectl -n tigera-operator get pods
```

4. Apply Calico custom resources (operator-native)

Create a file named `bootstrap/calico.yaml`:

```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
name: default
spec:
calicoNetwork:
ipPools:
- name: default-ipv4-ippool
cidr: 10.10.250.0/24
blockSize: 26
encapsulation: VXLANCrossSubnet
natOutgoing: Enabled
nodeSelector: all()

apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
name: default
spec: {}
```

Apply it:

```bash
kubectl apply -f bootstrap/calico.yaml
```

5. Wait for Calico to become ready

Monitor Calico components:

```bash
kubectl -n calico-system get pods
```

All pods should eventually be in `Running` state.

6. Restart workloads (one-time)

After Calico networking is active, restart pods so they receive Calico-managed IPs:

```bash
kubectl delete pods --all --all-namespaces
```

7. Verification

Confirm node and pod networking:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
```

You should see:

Pod IPs in `10.10.250.0/24`

No Flannel interfaces

Calico pods healthy
