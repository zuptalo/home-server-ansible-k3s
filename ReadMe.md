# Home Server With K3S

- Make sure the user that you will be using to connects to the remote host has proper access to run sudo command without password.
- SSH to the remote machine and run command below.
    ```
    sudo visudo
    ```
- Replace `username` in the line below with your server's actual username and add it to the end of the file which was opened after running previous command, save and exit.
    ```
    username ALL=(ALL) NOPASSWD:ALL
    ```

- Add your SSH public key to the remote host, so you can connect without entering password. 
  - Check if you already have a key**
    ```shell
    ls ~/.ssh/id_rsa.pub
    ```
  - If you don't have the file, generate one using command below.
    ```shell
    ssh-keygen
    ```
  - If you do have the file, add it to the remote host running command below.
    ```shell
    ssh-copy-id username@host
    ```
---

## Install KUBECTL, KUSTOMIZE and K9S on your local machine

- Install kubectl and kustomize on your local machine if you don't have them already.
    ```shell
    brew install kubectl kustomize derailed/k9s/k9s
    ``` 

---
## Installing K3S on your remote machine

- Update the [inventory](ansible%2Finventory) file and replace `home-server.example.com` with your server's hostname or IP address.
- Update the [inventory](ansible%2Finventory) file and replace `ubuntu` with your username on the server.
- If you are working inside your LAN, use the ip address of your server.
    ```shell
    nano ./ansible/inventory
    ```

- Run the ansible playbook below to install K3S on the specified host in your [inventory](ansible%2Finventory) file and get the config file added to ~/.kube/config.example on your local machine
  ```shell
  ansible-playbook -i ./ansible/inventory ./ansible/install-k3s.yaml
  ```

---

## Setting up the newly created K3S cluster

### Install Cert Manager
- IMPORTANT: Run this one twice with 30 seconds delay in between the runs!
```shell
kustomize build kubernetes/infrastructure/cert-manager | kubectl apply -f -
```
---
### Install Nginx Proxy Manager
- Make sure to replace `nginx.example.com` with the hostname you have picked for this service in its [ingress.yaml](kubernetes%2Finfrastructure%2Fnginx-proxy-manager%2Fingress.yaml)
- The hostname needs to be pointing to your home's public IP address
- Port 80 and 443 need to be forwarded to your home-server's internal ip address inside your router.
```shell
kustomize build kubernetes/infrastructure/nginx-proxy-manager | kubectl apply -f -
```
---
### Install Home Assistant
- Make sure to replace `home-assistant.example.com` with the hostname you have picked for this service in its [ingress.yaml](kubernetes%2Fapps%2Fhome-assistant%2Fingress.yaml)
- The hostname needs to be pointing to your home's public IP address
- Port 80 and 443 need to be forwarded to your home-server's internal ip address inside your router.
```shell
kustomize build kubernetes/apps/home-assistant | kubectl apply -f -
```

---

# Danger Zone!

### Delete Home Assistant
```shell
kustomize kubernetes/apps/home-assistant | kubectl delete -f -
```

### Delete Nginx Proxy Manager
```shell
kustomize build kubernetes/infrastructure/nginx-proxy-manager | kubectl delete -f -
```

### Delete Cert Manager
```shell
kustomize build kubernetes/infrastructure/cert-manager | kubectl delete -f -
```

### Uninstall K3s and dependencies
  ```shell
  ansible-playbook -i ./ansible/inventory ./ansible/uninstall-k3s.yaml
  ```
---
