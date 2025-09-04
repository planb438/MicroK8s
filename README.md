You need to assign roles to your nodes to make this a proper cluster. Here's how to fix it:

1. Remove the current nodes from the cluster (optional but clean)
First, drain and remove the worker nodes:

bash
microk8s kubectl drain worker1 --ignore-daemonsets --delete-emptydir-data
microk8s kubectl drain worker2 --ignore-daemonsets --delete-emptydir-data
microk8s kubectl delete node worker1 worker2
2. Set up the control plane node
On the cp node, run:

bash
microk8s kubectl label node cp node-role.kubernetes.io/control-plane=
microk8s kubectl taint node cp node-role.kubernetes.io/control-plane:NoSchedule
3. Join the worker nodes properly
Get the join command from the control plane node:

bash
microk8s add-node
This will give you a command like:

text
microk8s join 192.168.1.100:25000/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Run this command on both worker1 and worker2.

4. Label the worker nodes
After joining, label the worker nodes:

bash
microk8s kubectl label node worker1 node-role.kubernetes.io/worker=
microk8s kubectl label node worker2 node-role.kubernetes.io/worker=
5. Verify the setup
bash
microk8s kubectl get nodes -o wide
You should now see proper roles assigned:

text
NAME      STATUS   ROLES            AGE     VERSION
cp        Ready    control-plane    51m     v1.32.3
worker1   Ready    worker           3m43s   v1.32.3
worker2   Ready    worker           24s     v1.32.3
6. Optional: Enable necessary MicroK8s addons
bash
microk8s enable dns storage dashboard
7. Configure pod placement
Create tolerations and node selectors in your deployments to ensure:

Control plane pods stay on control plane nodes

Application pods go to worker nodes

Example deployment for worker nodes:

yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: ""
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: my-app
        image: nginx
This setup will ensure that critical control plane components stay on the control plane node while application workloads are scheduled on the worker nodes.
