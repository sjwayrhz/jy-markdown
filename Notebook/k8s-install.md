kubeadm join 192.168.176.31:6443 --token 1qhc5d.btnvctno70qvsh4d --discovery-token-ca-cert-hash sha256:0ead6d4d6aa8d4fc9d4418fce045316b580407c67cd67f51d215f7da03e3640d --control-plane --certificate-key 166b8e8e5c0ce997227de78eb48839cd0b2792c1676200509c5ee4927519f337


REGISTRY="kubeovn"
VERSION="v1.8.2"
IMAGE_PULL_POLICY="IfNotPresent"
POD_CIDR="10.4.0.0/16"                # Do NOT overlap with NODE/SVC/JOIN CIDR
POD_GATEWAY="10.4.0.1"
SVC_CIDR="10.6.0.0/16"                # Do NOT overlap with NODE/POD/JOIN CIDR
JOIN_CIDR="10.5.0.0/16"              # Do NOT overlap with NODE/POD/SVC CIDR