# Container registry and Containerlab

Containerlab is better when used with a container registry. No one loved to witness the uncontrolled proliferation of unversioned disk image (qcow2, vmdk) files shared via ftps, one drives and IM attachments.

We can do better!

Since containerlab deals with container images, it is natural to use a container registry to store them. Versioned, immutable, tagged and easily shareable with granular access control.

Whether you choose to use one of the public registries or a run a private one, the workflow is the same. Let's see what it looks like.

## Harbor registry

In this workshop we make use of an open-source registry called [Harbor](https://goharbor.io/). It is a CNCF graduated project and is a great choice for a private registry.

The registry has been already deployed in the workshop environment, but it is quite easy to deploy yourself in your own organization. It is a single docker compose stack that can be deployed in a few minutes.

The Harbor registry offers a neat Web UI to browse the registry contents, manage users and tune access control. You can log in to the registry UI like this:

<https://registry.wrkshpz.net>

using the `autoconuser` user and the password available in your workshop handout.

Managing the harbor registry is out of the scope of this workshop.

## Pushing images to the registry

***FOR REFERENCE ONLY***

This section is for reference only and you are not required to do this on your VM.

### 1 Logging in to the registry

To be able to push and pull the images from the workshop's registry, you need to login to the registry.

```bash
docker login registry.wrkshpz.net
```

```
# username: autoconuser
# password: as per your workshop handout
```

### 2 Listing local images

First, we need to identify the name of the image that we want to push to the registry. By listing the images in the local image store we can reliably identify the name of the image that we want to push.

```
docker images
```

On your system you will see a list of images, among which you will see:

```
REPOSITORY                      TAG          IMAGE ID       CREATED         SIZE
vrnetlab/sonic_sonic-vs         202405       9b419b0a2acf   2 weeks ago     6.37GB
```

This is the image that we built before and that we want to push to the registry so that next time we want to use it we won't have to build it again.

The image name consists of two parts:

- `vrnetlab/sonic_sonic-vs` - the repository name
- `202405` - the tag

Catenating these two parts together we get the full name of the image that we want to push to the registry.

### 3 Pushing the image to the registry

This section is for reference only and you are not required to do this on your VM.

Now that we know the name of the image that we want to push to the registry, we can push it.

We will use `docker push` to upload the image to the registry. Before this, let's tag the image with the name of the registry and a tag.

```bash
docker tag vrnetlab/sonic_sonic-vs:202405 registry.wrkshpz.net/autocon2/sonic-vs:202405
```

Now we can push the image to the registry.

```bash
docker push registry.wrkshpz.net/autocon2/sonic-vs:202405
```

Expected output:

```bash
The push refers to repository [registry.wrkshpz.net/autocon2/sonic-vs]
626d14695d27: Pushed 
da133bfdd77f: Pushed 
2e873d5d18ac: Pushed 
0595fbac089d: Pushed 
c3548211b826: Pushed 
latest: digest: sha256:77179add9a22a308b675f6eb5956f01feba25cf15f07cda9e8fb36784881b96e size: 1371
```

## Listing images from the registry

Once the image is copied, you can see it in the registry UI.

![pic](harbor-sonic.jpg)

## Using images from the registry

The whole point of pushing the image to the registry is to be able to use it in the future yourself and also to share it with others.

We will be using a SR Linux image 24.7.2 image that is already in the registry. This can be viewed in Harbor registry.

![pic](harbor-srl.jpg)

We can modify the `20-vm.clab.yml` file to make use of it:

```bash
name: vm
 
topology:
  nodes:
    sonic:
      kind: sonic-vm
      image: vrnetlab/sonic_sonic-vs:202405
    srl:
      kind: nokia_srlinux
      image: registry.wrkshpz.net/autocon2/nokia_srlinux:24.7.2

  links:
    - endpoints: ["sonic:eth1", "srl:e1-1"]
```

Deploy the lab using:

```bash
cd ~/ac2-clab/20-vm
sudo clab dep -t 20-vm.clab.yml
```

Expected output:

```bash
INFO[0000] Containerlab v0.59.0 started                 
INFO[0000] Parsing & checking topology file: 20-vm.clab.yml 
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="3fff:172:20:20::/64", MTU=1500 
INFO[0000] Pulling registry.wrkshpz.net/autocon2/nokia_srlinux:24.7.2 Docker image 
INFO[0036] Done pulling registry.wrkshpz.net/autocon2/nokia_srlinux:24.7.2 
INFO[0036] Creating lab directory: /home/autoconuser/clab-vm-reg 
INFO[0036] Creating container: "sonic"                  
INFO[0036] Creating container: "srl"                    
INFO[0037] Running postdeploy actions for Nokia SR Linux 'srl' node 
INFO[0037] Created link: sonic:eth1 <--> srl:e1-1       
INFO[0053] Adding containerlab host entries to /etc/hosts file 
INFO[0053] Adding ssh config for containerlab nodes     
+---+-------------------+--------------+----------------------------------------------------+---------------+---------+----------------+----------------------+
| # |       Name        | Container ID |                       Image                        |     Kind      |  State  |  IPv4 Address  |     IPv6 Address     |
+---+-------------------+--------------+----------------------------------------------------+---------------+---------+----------------+----------------------+
| 1 | clab-vm-reg-sonic | bc6135bae8c5 | vrnetlab/sonic_sonic-vs:202405                     | sonic-vm      | running | 172.20.20.3/24 | 3fff:172:20:20::3/64 |
| 2 | clab-vm-reg-srl   | a3cef13bd4b4 | registry.wrkshpz.net/autocon2/nokia_srlinux:24.7.2 | nokia_srlinux | running | 172.20.20.2/24 | 3fff:172:20:20::2/64 |
+---+-------------------+--------------+----------------------------------------------------+---------------+---------+----------------+----------------------+
```


Not only this gives us an easy way to share images with others, but also it enables stronger reproducibility of the lab, as the users of our lab would use exactly the same image that we built.
