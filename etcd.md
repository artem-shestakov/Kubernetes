# `Etcd` setup
## Systemd file example
```shell
NODE_IP="192.168.56.31"

ETCD_NAME=$(hostname -s)

ETCD1_IP="192.168.56.31"
ETCD2_IP="192.168.56.32"
ETCD3_IP="192.168.56.33"


cat <<EOF >/etc/systemd/system/etcd.service
[Unit]
Description=etcd

[Service]
Type=exec
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --initial-advertise-peer-urls http://${NODE_IP}:2380 \\
  --listen-peer-urls http://${NODE_IP}:2380 \\
  --advertise-client-urls http://${NODE_IP}:2379 \\
  --listen-client-urls http://${NODE_IP}:2379,http://127.0.0.1:2379 \\
  --initial-cluster-token etcd-cluster-1 \\
  --initial-cluster etcd01=http://${ETCD1_IP}:2380,etcd02=http://${ETCD2_IP}:2380,etcd03=http://${ETCD3_IP}:2380 \\
  --initial-cluster-state new
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable etcd

ETCDCTL_API=3 etcdctl --endpoints=http://127.0.0.1:2379 member list
```

## `Etcd` with `TLS`
>Cfssl tool https://github.com/cloudflare/cfssl
>Example https://github.com/coreos/docs/blob/master/os/generate-self-signed-certificates.md
* Create CA
```shell
cat > ca-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
            "etcd": {
                "expiry": "8760h",
                "usages": ["signing","key encipherment","server auth","client auth"]
            }
        }
    }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "My etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "RU",
      "L": "RU",
      "O": "Kubernetes",
      "OU": "etcd",
      "ST": "Moscow"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
* Create server certs
User default config file:
```shell
cfssl print-defaults csr > server.json
```
or create new:
```shell
ETCD1_IP="192.168.56.31"
ETCD2_IP="192.168.56.32"
ETCD3_IP="192.168.56.33"

cat > etcd.json <<EOF
{
  "CN": "etcd",
  "hosts": [
    "localhost",
    "127.0.0.1",
    "${ETCD1_IP}",
    "${ETCD2_IP}",
    "${ETCD3_IP}"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "RU",
      "L": "RU",
      "O": "Kubernetes",
      "OU": "etcd",
      "ST": "Moscow"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd.json | cfssljson -bare etcd
```
* Copy the certificates to etcd nodes
rm -f /etc/etcd/pki/*
cp /vagrant/ca.pem /vagrant/etcd* /etc/etcd/pki/

* Copy the certificates to a standard location on servers
```shell
mkdir -p /etc/etcd/pki
mv ca.pem etcd.pem etcd-key.pem /etc/etcd/pki/ 
```
* Setup Systemd file
```shell
NODE_IP="192.168.56.31"

ETCD_NAME=$(hostname -s)

ETCD1_IP="192.168.56.31"
ETCD2_IP="192.168.56.32"
ETCD3_IP="192.168.56.33"


cat <<EOF >/etc/systemd/system/etcd.service
[Unit]
Description=etcd

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/pki/etcd.pem \\
  --key-file=/etc/etcd/pki/etcd-key.pem \\
  --peer-cert-file=/etc/etcd/pki/etcd.pem \\
  --peer-key-file=/etc/etcd/pki/etcd-key.pem \\
  --trusted-ca-file=/etc/etcd/pki/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/pki/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${NODE_IP}:2380 \\
  --listen-peer-urls https://${NODE_IP}:2380 \\
  --advertise-client-urls https://${NODE_IP}:2379 \\
  --listen-client-urls https://${NODE_IP}:2379,https://127.0.0.1:2379 \\
  --initial-cluster-token etcd-cluster-1 \\
  --initial-cluster etcd01=https://${ETCD1_IP}:2380,etcd02=https://${ETCD2_IP}:2380,etcd03=https://${ETCD3_IP}:2380 \\
  --initial-cluster-state new
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
## Live migration from `HTTP` to `HTTPS`
1. Check members.
```shell
ETCDCTL_API=3 etcdctl --endpoints=http://127.0.0.1:2379 member list
2dc86728c292b9d1, started, etcd03, http://192.168.56.33:2380, https://192.168.56.33:2379, false
6e8939086e8ed47b, started, etcd02, http://192.168.56.32:2380, https://192.168.56.32:2379, false
6fdca8900360571e, started, etcd01, http://192.168.56.31:2380, http://192.168.56.31:2379, false
```
2. Generate certs and copy them to all nodes(Look `Etcd with TLS`).
3. Update members peer URL to `https` **except one member**
```shell
# Update 2 of 3 member
ETCDCTL_API=3 etcdctl --endpoints=http://127.0.0.1:2379 member update 6e8939086e8ed47b --peer-urls=https://192.168.56.32:2380
ETCDCTL_API=3 etcdctl --endpoints=http://127.0.0.1:2379 member update 2dc86728c292b9d1 --peer-urls=https://192.168.56.33:2380
```
4. Update systemd file of updated members(Look `Etcd with TLS`) and restart `etcd`. Check members.
```shell
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/etcd/pki/ca.pem   --cert=/etc/etcd/pki/etcd.pem   --key=/etc/etcd/pki/etcd-key.pem   member list
2dc86728c292b9d1, started, etcd03, https://192.168.56.33:2380, https://192.168.56.33:2379, false
6e8939086e8ed47b, started, etcd02, https://192.168.56.32:2380, https://192.168.56.32:2379, false
6fdca8900360571e, started, etcd01, http://192.168.56.31:2380, http://192.168.56.31:2379, false
```
5. Update last member.
```shell
ETCDCTL_API=3 etcdctl --endpoints=http://127.0.0.1:2379 member update 6fdca8900360571e --peer-urls=https://192.168.56.31:2380
Member 6fdca8900360571e updated in cluster e31a7a53107a18e5
```
6. Update systemd file of last `http` member and restart `etcd`.

## External `Etcd` and `K8s`
1. Copy certs to `k8s` hosts
```shell
mkdir -p /etc/kubernetes/pki/etcd
cp <certs> /etc/kubernetes/pki/etcd/
```
2. Create config file `kube-config.yml`
>Use own `localAPIEndpoint` address on each node
```yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.56.11
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
controlPlaneEndpoint: "k8s-api.local:6443"
networking:
  podSubnet: 192.168.0.0/16
etcd:
  external:
    endpoints:
    - "https://192.168.56.31:2379"
    - "https://192.168.56.32:2379"
    - "https://192.168.56.33:2379"
    caFile: "/etc/kubernetes/pki/etcd/ca.pem"
    certFile: "/etc/kubernetes/pki/etcd/etcd.pem"
    keyFile: "/etc/kubernetes/pki/etcd/etcd-key.pem"
```
3. Init cluster
```shell
kubeadm init --config kube-config.yml --upload-certs | tee kubeadm_init.log
```
4. Join members
```shell
kubeadm join k8s-api.local:6443 --token qropb6.g9psd9eru7m8w8qn \
        --discovery-token-ca-cert-hash sha256:c813f797fa0669b5f10bb03f2edabc644bfbaddeeacdf82ff3b4ddea06b10554 \
        --control-plane --certificate-key c67b6eba9cba27f4afd2933386a48056a80df62a85a2978d1f58516e57bf6c11 \
        --apiserver-advertise-address <node_ip_address>
```