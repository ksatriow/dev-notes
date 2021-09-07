# **Kubernetes**

```shell
sudo su -

swapoff -a

kubectl version

LOCAL_IP=$(ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1' | tail -n1);
echo $LOCAL_IP;

kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=$LOCAL_IP


kubeadm config images pull

systemctl status kubelet

journalctl -xeu kubelet



kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

curl http://10.0.2.15:6443/api?timeout=32s:




kubeadm token list

kubeadm token generate

kubectl get pods --all-namespaces

kubeadm join --token $TOKEN $LOCAL_IP:6443 \
  --discovery-token-unsafe-skip-ca-verification

kubectl get nodes
```

### **run a container, verify**

```shell
kubectl run --image=nginx nginx-app \
  --port=80 \
  --env="DOMAIN=cluster"

docker ps -a | grep nginx

kubectl get deployments nginx-app

kubectl describe deployments nginx-app

# make the container accessible
kubectl expose deployment nginx-app --port=80 --name=nginx-http

kubectl get services nginx-http

kubectl describe services nginx-http

kubectl get pods --output=wide
```

### **delete service**

```shell
kubectl delete services nginx-http
```

### **delete deployment**

```shell
kubectl delete deployment nginx-app
```

### **Ansible step, wait for kubectl to startup**

```shell
- name: Wait for kubectl to become available
  wait_for:
    port: 6443
    delay: 5
```
