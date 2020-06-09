# microk8s
https://microk8s.io/

### install
sudo snap install microk8s --classic --channel=1.17/stable

### alias setup at bash
alias mkctl="microk8s kubectl

### enable dns
microk8s enable dns

### enable dashboard
microk8s enable dashboard

### check status
microk8s status --wait-ready

### start / stop
microk8s start
microk8s stop

