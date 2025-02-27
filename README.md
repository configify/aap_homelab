# AAP home lab

Deploy your own LAB with Ansible Automation Platform, ansible-core dev server and test RHEL host on a single laptop in Docker containers

Bonus: Mailhog


## Create network
```
docker network create lab
```


## Deploy test host with RHEL9.5

Deploy container with blank RHEL:
```
docker run -d --pull always --net lab --name host1 -h host1 --privileged quay.io/configify/homelab:lab-rhel9.5-base
```

Register with your subscription if required:
```
docker exec -it host1 /bin/bash 
```

```
su -
```

```
subscription-manager register --username=<your_RH_username>
```


## Deploy dev box with RHEL9.5

Deply container for development with ansible-core:
```
docker run -d --pull always --net lab --name dev -h dev --privileged -v <local_folder_with_your_repo>:/ansible quay.io/configify/homelab:lab-rhel9.5-dev
```

Alternatively deploy a box that includes git, zsh, eza, bat and 'oh my zsh':
```
docker run -d --pull always --net lab --name dev -h dev --privileged -v <local_folder_with_your_repo>:/ansible quay.io/configify/homelab:lab-rhel9.5-fancy-dev
```

Check ansible version:
```
docker exec -it dev /bin/bash
```

```
su - kk
```

```
cd /ansible &&  
ansible --version
```

Check ssh connection to test host:
```
ssh host1
```

Install python modules, collections and run playbooks:
```
pip install <...>
ansible-galaxy collection install -r collections/requirements.yml
ansible-playbook <...>
```


## Deploy AAP cluster on RHEL9.5
```
docker run -d --pull always -p 4431:443 --net lab --name aap-controller.int -h aap-controller.int --privileged -v <local_folder_with_aap_bundle>:/aap quay.io/configify/homelab:lab-rhel9.5-controller 
```

```
docker run -d --pull always -p 4432:443 --net lab --name aap-hub.int -h aap-hub.int --privileged quay.io/configify/homelab:lab-rhel9.5-hub
```

```
docker run -d --pull always -p 4433:443 --net lab --name aap-gateway.int -h aap-gateway.int --privileged quay.io/configify/homelab:lab-rhel9.5-hub
```

Controller - register with your subscription:
```
docker exec -it aap-controller.int /bin/bash
```

```
su -
```

```
subscription-manager register --username=<your_RH_username>
```

Controller - install and enable Chrony:
```
dnf install chrony
```

```
systemctl enable --now chronyd
```

```
exit
```

```
su - kk
```

Hub - register with subscription:
```
ssh aap-hub.int
```

```
sudo su -
```

```
subscription-manager register --username=<your_RH_username>
```

Hub - install and enable Chrony:
```
dnf install chrony
```

```
systemctl enable --now chronyd
```

```
exit
exit
```

Gateway - register with your subscription:
```
ssh aap-gateway.int
```

```
sudo su -
```

```
subscription-manager register --username=<your_RH_username>
```

Gateway - install and enable Chrony:
```
dnf install chrony
```

```
systemctl enable --now chronyd
```

```
exit
exit
```

Test ssh connection to Controller:
```
ssh aap-controller.int
```

```
exit
```

Controller - Install AAP:
```
cd /aap
```

```
tar -xzf <latest_AAP_setup_bundle>
```

```
cd <path_it_extracted_to>
```

```
cp inventory inventory_bckp
```

```
vi inventory
```

<... edit inventory file as required ...>


Run AAP setup:
```
./setup.sh
```

```
exit
exit
```

Access Gateway GUI: https://localhost:4433


## Deploy Mailhog for testing emails
```
docker run -d -p 8025:8025 --net lab --name mail -h mail mailhog/mailhog
```

Access interface: http://localhost:8025
