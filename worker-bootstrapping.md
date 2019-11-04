# Worker Nodes Bootstrapping

## Download and Install Certificates and Config Files

```bash
HOSTNAME=$(hostname -s)
sudo mkdir -p /var/lib/kubernetes
sudo mkdir -p /var/lib/kubelet
sudo aws s3 cp s3://gresik/certificates/kubelet/${HOSTNAME}.pem /var/lib/kubelet/kubelet.pem
sudo aws s3 cp s3://gresik/certificates/kubelet/${HOSTNAME}-key.pem /var/lib/kubelet/kubelet-key.pem
sudo aws s3 cp s3://gresik/certificates/kubelet/${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo aws s3 cp s3://gresik/certificates/ca/ca.pem /var/lib/kubernetes/ca.pem
```

## Download and Install Kubernetes Binaries

```bash
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
  
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```