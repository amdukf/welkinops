# Kubernetes TLS/Certificate issues and renew

#### Use this when cluster has TLS / certificate issues


```
kubeadm certs check-expiration

ls -l /etc/kubernetes/pki
ls -l /etc/kubernetes/pki/etcd

crictl ps | grep kube
crictl ps -a | grep kube
crictl ps -a | grep etcd

systemctl restart kubelet

# Run if needed, be carefull
crictl rm -f $(crictl ps -aq)

kubectl get nodes --kubeconfig=/etc/kubernetes/admin.conf
kubectl get nodes
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide

kubectl logs -n kube-system <pod-name>

kubeadm certs renew all

# If needed, when we write renew all, it will renew all of the certs
kubeadm certs renew apiserver
kubeadm certs renew apiserver-kubelet-client
kubeadm certs renew front-proxy-client

cp /etc/kubernetes/admin.conf ~/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```
