# pv_migration_nginx_k3s_vagrant_libvirt_ansible

Vagrant-libvirt setup that creates a VM with k3s.

Default OS is openSUSE Leap 15.5, but that can be changed in the Vagrantfile.
Please be aware, that this might break the Ansible provisioning.

The setup automatically creates two PVCs in the `default` namespace and installs
two instances of the [Bitnami Nginx helm
chart](https://artifacthub.io/packages/helm/bitnami/nginx), each using one of
the PVCs.

## Getting started

1. You need `vagrant`, obviously. And `git`. And Ansible...
1. Fetch the box, per default this is `opensuse/Leap-15.5.x86_64`, using
   `vagrant box add opensuse/Leap-15.5.x86_64`.
1. Make sure the git submodules are fully working by issuing
   `git submodule init && git submodule update`
1. Run `vagrant up`
1. Run `kubectl --kubeconfig ansible/k3s-kubeconfig get nodes` and you should
   see your server.

The PVCs are being created, but only `nginx-pvc1` contains a `index.html`. So if
you try to reach the `nginx-pvc2` Nginx server via the Ingress, you will get an
error 403.

```bash
$ curl -s http://nginx-pvc1.192.0.2.13.sslip.io
vagrant-libvirt
$ curl -s http://nginx-pvc2.192.0.2.13.sslip.io
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx</center>
</body>
</html>
$
```

(Replace `192.0.2.13` with your VM's IP address)

## Migration of a PV's contents

You can use [pv-migrate](https://github.com/utkuozdemir/pv-migrate) to migrate
the content of one volume to another. In this setup, you can transfer the
`index.html` from `nginx-pvc1` to `nginx-pvc2`.

```bash
pv-migrate \
    --source-kubeconfig ansible/k3s-kubeconfig \
    --dest-kubeconfig ansible/k3s-kubeconfig \
    migrate \
    --ignore-mounted \
    nginx-pvc1 nginx-pvc2
```

If you `curl` the `nginx-pvc2` ingress's address while running this command, you
will see a change in the output once the `index.html` was transferred:

```bash
watch curl -s http://nginx-pvc2.192.0.2.13.sslip.io
```

(Replace `192.0.2.13` with your VM's IP address)

## Cleaning up

The VMs can be torn down after playing around using `vagrant destroy`. This will
also remove the kubeconfig file `ansible/k3s-kubeconfig`.
