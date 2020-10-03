# rpi-k3s-bootstrap

Ansible tooling to setup a small raspberry pi kubernetes cluster with k3s.
Fast ⚡️

## Not included

These steps assume you have a booted raspberry pi running [raspbian buster](https://www.raspberrypi.org/downloads/raspbian/) and [ssh enabled](https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/) attached to your computer via ethernet cable.

For minimum hardware requirements (which rpi model), [please refer to k3s](https://rancher.com/docs/k3s/latest/en/installation/installation-requirements/#hardware).

## Before you begin

Rename the inventory file:

`mv ./bootstrap/inventory.template.yml ./bootstrap/inventory.yml`

Add the correct path to your ssh keys for following variables:

- `rpi_user_pub_key_content`
- `ansible_ssh_private_key_file`

## 01-rpi-base

This playbook will create a secure password-less sudo user for your raspberry pi as well as setting a custom hostname and static IPv4 address.
Once done, it will shutdown the rpi.

```
ansible-playbook \
-i ./bootstrap/inventory.yml \
./bootstrap/01-rpi-base/playbook.yml
```

## attach to network

You can now attach your rpi directly to your router via ethernet cable.
Once is powered on you should be able to access it via (adapt user/host if you changed them in inventory):


```
ssh bot@k3s-master-1.local
```


## 02-rpi-k3s

You can now run the second playbook to install k3s on your host.

```
ansible-playbook \
-i ./bootstrap/inventory.yml \
./bootstrap/02-rpi-k3s/playbook.yml
```

This will prompt two variables:

- the target host (`k3s-master-1.local` in the provided template)
- if the target host is a master node (control plane)

The playbook will take a bit of time depending on your device and network latency.

Once done, you can check that everything works by running the following command after you logged in to you pi.

```
kubectl -n kube-system get pods
```

The output should look something like:

```
NAME                                      READY   STATUS      RESTARTS   AGE
local-path-provisioner-58fb86bdfd-xl64x   1/1     Running     0          117m
metrics-server-6d684c7b5-xzg4c            1/1     Running     0          117m
helm-install-traefik-ngqvv                0/1     Completed   1          117m
coredns-6c6bb68b64-rjsb9                  1/1     Running     0          117m
svclb-traefik-kwlbs                       2/2     Running     0          116m
traefik-7b8b884c8-7p6h6                   1/1     Running     0          116m
svclb-traefik-4kf78                       2/2     Running     2          83m
```

## further steps

Now your master node is up and running, you can proceed to bootstrap as many worker nodes as you need, by repeating the steps above.

Important here is to change following variables in your inventory (you can find details in the comments):

- `rpi_hostname`
- `rpi_dhcpcd_ip_address`
- `k3s_master_ip`
- `k3s_master_token`

### check that the worker is setup correctly

Once you repeated the steps above for your worker node, you can run following command in the master node:

```
kubectl get nodes
```

Which should print something like:

```
NAME           STATUS   ROLES    AGE    VERSION
k3s-master-1   Ready    master   121m   v1.17.4+k3s1
k3s-worker-1   Ready    <none>   86m    v1.17.4+k3s1
```

## 03-rpi-files

At this point, you probably want to retrieve the k3s master kubeconfig from your master node.

You cna do so by running:

```
ansible-playbook \
-i ./bootstrap/inventory.yml \
./bootstrap/03-rpi-files/playbook.yml
```

This will pull to you local machine the k3s config as well as the rpi pub key.