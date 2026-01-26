# Flannel

If using flannel

### Download the yaml 
```shell
curl -LO https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
```

### Modify the following

#### Modify cidr

```shell
net-conf.json: |
    {
      "Network": "<replace-with-cidr-used-during-kubeadmin-init>", # Update this to match the custom PodCIDR
      "Backend": {
        "Type": "vxlan"
      }
```

#### Modify gateway

```shell
  args:
  - --ip-masq
  - --kube-subnet-mgr
  - --iface=<ethernet-adapter>
```