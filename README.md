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