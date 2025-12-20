https://docs.tigera.io/calico/latest/getting-started/kubernetes/k3s/quickstart
```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=192.168.0.0/16 --disable-network-policy --disable=traefik" sh -
```
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
```
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/operator-crds.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/tigera-operator.yaml
```
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/custom-resources.yaml
```
```bash
nano calico-ip-pool.yaml
```
```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: k3s-pool
spec:
  cidr: 10.10.250.128/25
  ipipMode: Always
  natOutgoing: true
  disabled: false
```
```bash
kubectl delete ippool default-ipv4-ippool
kubectl apply -f calico-ip-pool.yaml
```
```bash
kubectl delete pods --all --all-namespaces
```
```bash
kubectl annotate node smart.home.lab projectcalico.org/ipv4pools='["k3s-ip-pool"]'
```
