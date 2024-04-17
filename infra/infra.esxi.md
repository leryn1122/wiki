---
id: infra.esxi
tags:
- esxi
- infra
- virt
- vm
title: ESXi & vSphere

---


# ESXi & vSphere
vSphere 是 VMWare 整个套件的集合，它包括了 ESXi，vCenter 等等。


```bash
wget http://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-amd64.img

virt-customize -a ubuntu-22.04-server-cloudimg-amd64.img --root-password password:ubuntu
```
