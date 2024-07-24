Perform a version upgrade on a Kubernetes cluster using KubeADM
Solution
Doc: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

After installing Kubernetes v1.27 here: install

We will now upgrade the cluster to v1.28.

On controlplane node:

# Add 1.28 repository
sudo sh -c 'echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" >> /etc/apt/sources.list.d/kubernetes.list'

# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.28.1-1.1
sudo apt-mark hold kubeadm

# Upgrade controlplane node
kubectl drain k8s-controlplane --ignore-daemonsets
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.28.1

# Update Flannel
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Make controlplane node reschedulable
kubectl uncordon k8s-controlplane
On worker nodes:

# Add 1.28 repository
sudo sh -c 'echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" >> /etc/apt/sources.list.d/kubernetes.list'

# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.28.1-1.1
sudo apt-mark hold kubeadm

# Upgrade the other node
kubectl drain k8s-node-1 --ignore-daemonsets
sudo kubeadm upgrade node

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Make worker node reschedulable
kubectl uncordon k8s-node-1
Verify that the nodes are upgraded to v1.28.1:

kubectl get nodes
NAME               STATUS                     ROLES           AGE   VERSION
k8s-controlplane   Ready                      control-plane   15m   v1.28.1
k8s-node-1         Ready,SchedulingDisabled   <none>          13m   v1.28.1
k8s-node-2         Ready,SchedulingDisabled   <none>          13m   v1.28.1
Facilitate operating system upgrades
Solution
When having a one controlplane node in you cluster, you cannot upgrade the OS system (with reboot) without loosing temporarily access to your cluster.

Here we will upgrade our worker nodes:

# Hold kubernetes from upgrading
sudo apt-mark hold kubeadm kubelet kubectl

# Upgrade node
kubectl drain k8s-node-1 --ignore-daemonsets
sudo apt update && sudo apt upgrade -y # Be careful about container runtime (e.g., docker) upgrade.

# Reboot node if necessary
sudo reboot

# Make worker node reschedulable
kubectl uncordon k8s-node-1
