---

sudo snap install microk8s --classic --channel=1.28

--

sudo microk8s status --wait-ready

--
sudo microk8s enable dns

--

sudo microk8s enable storage

--

sudo microk8s enable metallb

--

sudo microk8s enable dashboard

--

sudo microk8s enable ingress

--

Troubleshooting Commands
bash
# Check MicroK8s status
sudo microk8s status

# Check addons
sudo microk8s addons list

# Check community status
sudo snap get microk8s community

# Restart MicroK8s
sudo microk8s stop
sudo microk8s start

# Reset if needed
sudo microk8s reset
