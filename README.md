# 4G-core-mme-loadbalancer

Repo is to show how an IPVS LB can be used for multiple MMEs in a 4G core.

Summary of software used:

- KIND kubernetes
- Open5gs 4G core using Helm Chart
- Calico CNI
- IPVS Loadbalancer running in docker container
- srsRAN simulator running in docker container

KIND deployment:

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.16.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

sudo kind create cluster -n open5gs-4g-core --config=/tmp/kind-config.yaml

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml

kubectl create -f kind-manifest/custom-resources.yaml

kubectl create ns open5gs

kubectl create -f open5gs-helm/mongodb/

kubectl -n open5gs create secret generic diameter-ca --from-file=open5gs-helm/tls-certs/cacert.pem

kubectl -n open5gs create secret tls hss-tls \
  --cert=open5gs-helm/tls-certs/hss.cert.pem \
  --key=open5gs-helm/tls-certs/hss.key.pem
  
kubectl -n open5gs create secret tls mme-tls \
  --cert=open5gs-helm/tls-certs/mme.cert.pem \
  --key=open5gs-helm/tls-certs/mme.key.pem

kubectl -n open5gs create secret tls pcrf-tls \
  --cert=open5gs-helm/tls-certs/pcrf.cert.pem \
  --key=open5gs-helm/tls-certs/pcrf.key.pem

kubectl -n open5gs create secret tls smf-tls \
  --cert=open5gs-helm/tls-certs/smf.cert.pem \
  --key=open5gs-helm/tls-certs/smf.key.pem

helm upgrade --install -n open5gs core4g open5gs-helm/

# Set POD route to go via KIND worker container IP
sudo ip route add 10.240.0.0/16 via 172.18.0.2

# Inside MME-LB container:
sysctl -w net.ipv4.vs.conntrack=1
iptables -t nat -A POSTROUTING -m ipvs --vaddr 172.18.0.103/32 --vport 36412 -j SNAT --to-source 172.18.0.103

iptables -t nat -A POSTROUTING -o eth0 --dst 10.240.216.73 -m ipvs --ipvs --vaddr 172.18.0.103 --vport 36412 --vmethod masq -j SNAT --to-source 172.18.0.103
iptables -t nat -A POSTROUTING -o eth0 --dst 10.240.216.74 -m ipvs --ipvs --vaddr 172.18.0.103 --vport 36412 --vmethod masq -j SNAT --to-source 172.18.0.103