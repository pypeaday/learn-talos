
Building off of [their docs](https://www.talos.dev/v1.8/introduction/getting-started/)

## CLI
nic in ~   (dev) 󰒄 󱔎 NO PYTHON ENVIORNMENT SET
✗ brew install siderolabs/tap/talosctl

The version of `talosctl` determines the version of talos that will be installed on the machine, _not_ the versioin on the iso used to initally boot the image.

### Hosts

I edited `/etc/hosts` after creating 2 VMs and getting their IPs.

```
# the rest of your hosts file

192.168.1.172  talos control.talos
192.168.1.173  talos worker.talos
```

nic in ~   (dev) 󰒄 󱔎 NO PYTHON ENVIORNMENT SET  took 13s
❯ talosctl gen config talos-clsuter https://control.talos:6443
generating PKI and tokens
Created /home/nic/controlplane.yaml
Created /home/nic/worker.yaml
Created /home/nic/talosconfig

nic in ~   (dev) 󰒄 󱔎 NO PYTHON ENVIORNMENT SET
✗ talosctl -n control.talos disks --insecure
DEV          MODEL          SERIAL   TYPE      UUID   WWID   MODALIAS                    NAME   SIZE     BUS_PATH                                                  SUBSYSTEM          READ_ONLY   SYSTEM_DISK
/dev/loop0   -              -        UNKNOWN   -      -      -                           -      75 MB    /virtual                                                  /sys/class/block   *
/dev/sr0     QEMU DVD-ROM   -        CD        -      -      scsi:t-0x05                 -      105 MB   /pci0000:00/0000:00:1f.2/ata1/host0/target0:0:0/0:0:0:0   /sys/class/block
/dev/vda     -              -        HDD       -      -      virtio:d00000002v00001AF4   -      54 GB    /pci0000:00/0000:00:02.3/0000:04:00.0/virtio3             /sys/class/block


## Setup

> edit the disks in config to not be /dev/sda because it might not be that disk - I needed /dev/vda

talosctl apply-config --insecure --nodes control.talos --file controlplane.yaml
talosctl apply-config --insecure --nodes worker.talos --file worker.yaml

Call this ONCE against ONE controlplaine node

talosctl bootstrap --nodes 192.168.1.172 --endpoints 192.168.1.172 --talosconfig=./talosconfig
I couldn't get this to work with the dns reference... talosctl bootstrap --nodes control.talos --endpoints control.talos --talosconfig=./talosconfig

Finally get the kube config and have it merged with `~/.kube/config`

talosctl kubeconfig -n 192.168.1.172 -e 192.168.1.172 --talosconfig=./talosconfig

Then using `kubecm` we can see it's merged appropriately.

```
❯ kubecm list  
+------------+------------------------+------------------+------------------------+-------------------------------+--------------+
|   CURRENT  |          NAME          |      CLUSTER     |          USER          |             SERVER            |   Namespace  |
+============+========================+==================+========================+===============================+==============+
|      *     |   admin@talos-cluster  |   talos-cluster  |   admin@talos-cluster  |   https://control.talos:6443  |    default   |
+------------+------------------------+------------------+------------------------+-------------------------------+--------------+
|            |       kind-foo-tf      |    kind-foo-tf   |       kind-foo-tf      |    https://127.0.0.1:36505    |    default   |
+------------+------------------------+------------------+------------------------+-------------------------------+--------------+

Cluster check succeeded!
Kubernetes version v1.31.2
Kubernetes master is running at https://control.talos:6443
[Summary] Namespace: 4 Node: 2 Pod: 6 
```

## Adding a node?

I made another vm, added the IP to my `/etc/hosts`

```
# the rest of your hosts file

192.168.1.172  talos control.talos
192.168.1.173  talos worker.talos worker1.talos
192.168.1.174  talos worker2.talos

```

talosctl apply-config --insecure --nodes worker2.talos --file worker.yaml

and after about a minute I saw it in my cluster in k9s

```
❯ kubectl get nodes 
NAME            STATUS   ROLES           AGE     VERSION
talos-50i-xih   Ready    <none>          9m21s   v1.31.2
talos-gzn-xo4   Ready    control-plane   9m22s   v1.31.2
talos-sff-al0   Ready    <none>          3m6s    v1.31.2  # the new one!
```