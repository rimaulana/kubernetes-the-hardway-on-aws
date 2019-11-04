# ETCD Bootstrapping

## Download and Install the ectd Binaries

```bash
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz"
```

Extract and install the `etcd` server and the `etcdctl` command line utility:

```bash
tar -xvf etcd-v3.4.0-linux-amd64.tar.gz
sudo mv etcd-v3.4.0-linux-amd64/etcd* /usr/local/bin/
```

## Download Certificates

```bash
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo aws s3 cp s3://gresik/certificates/etcd/etcd.pem /etc/etcd/
sudo aws s3 cp s3://gresik/certificates/etcd/etcd-key.pem /etc/etcd/
sudo aws s3 cp s3://gresik/certificates/kubernetes/kubernetes.pem /etc/etcd/
sudo aws s3 cp s3://gresik/certificates/kubernetes/kubernetes-key.pem /etc/etcd/
sudo aws s3 cp s3://gresik/certificates/ca/ca.pem /etc/etcd/
```

## Configure the etcd Server

```bash
INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
ETCD_NAME=$(hostname -s)
DNS_SUFFIX=gresik.io

cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/etcd.pem \\
  --peer-key-file=/etc/etcd/etcd-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${ETCD_NAME}.${DNS_SUFFIX}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${ETCD_NAME}.${DNS_SUFFIX}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster etcd-1=https://etcd-1.${DNS_SUFFIX}:2380,etcd-2=https://etcd-2.${DNS_SUFFIX}:2380,etcd-3=https://etcd-3.${DNS_SUFFIX}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Start the etcd Server

```bash
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

## Testing the config
```bash
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```