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

using the `admin` user and the password available in your workshop handout.

When logged in as `admin` you can created users, repositories, browse the registry contents and many more. Managing the harbor registry is out of the scope of this workshop.

## Pushing images to the registry

We will push one of the images that we've built in the previous section to the registry effectively saving it and making it available and reusable to other users who have access to this registry.

### 1 Logging in to the registry

To be able to push and pull the images from the workshop's registry, you need to login to the registry.

```bash
docker login registry.wrkshpz.net
```

```
# username: admin
# password: as per your workshop handout
```

### 2 Listing local images

First, we need to identify the name of the image that we want to push to the registry. By listing the images in the local image store we can reliably identify the name of the image that we want to push.

```
docker images
```

On your system you will see a list of images, among which you will see:

```
REPOSITORY              TAG         IMAGE ID       CREATED       SIZE
vrnetlab/nokia_sros     24.7.R1     d45128fc2914   2 hours ago   889MB
```

This is the image that we built before and that we want to push to the registry so that next time we want to use it we won't have to build it again.

The image name consists of two parts:

- `vrnetlab/nokia_sros` - the repository name
- `24.7.R1` - the tag

Catenating these two parts together we get the full name of the image that we want to push to the registry.

### 3 Pushing the image to the registry

Now that we know the name of the image that we want to push to the registry, we can push it. Usually you will see a sequence of `docker tag` and `docker push` commands being executed, but we are cool kids here so we will do it in one go.

Using the skopeo tool - <https://github.com/containers/skopeo> - we can push the image to the registry in one go. The command to use has the following format:

```
skopeo copy docker-deamon://<local image name> docker://<registry>/<repository>:<tag>
```

Since everyone would want to push their own image to the registry we will need to append a user id to the tag name so that you won't overwrite each other's images. Like the below command pushes the image to the user with ID=1:

```bash
# note the appended -1 at the end of the tag
skopeo copy \
docker-daemon:vrnetlab/nokia_sros:24.7.R1 \
docker://registry.wrkshpz.net/library/nokia_sros:24.7.R1-1
```

## Listing images from the registry

Once the image is copied, you can see it in the registry UI.

![pic](https://gitlab.com/rdodin/pics/-/wikis/uploads/3f3d08696dd6bb83cf6e223a5f8f6c39/image.png)

If you want to get the list of available repositories/tags in the registry, you can use registry API and skopeo.

Listing available repositories:

```bash
 curl -s -u 'admin:nokia2024' https://registry.wrkshpz.net/v2/_catalog | jq
{
  "repositories": [
    "admin/nokia_srl",
    "library/nokia_srl"
  ]
}
```

Listing available tags for a given repository:

```bash
skopeo list-tags docker://registry.wrkshpz.net/library/nokia_srl
{
    "Repository": "registry.wrkshpz.net/library/nokia_srl",
    "Tags": [
        "24.7.1"
    ]
}
```

## Using images from the registry

The whole point of pushing the image to the registry is to be able to use it in the future yourself and also to share it with others. And now that we have the image in the registry, we can modify the `20-vm.clab.yml` file to make use of it:

```diff
name: vm
 
topology:
  nodes:
    sonic:
      kind: sonic-vm
      image: vrnetlab/sonic_sonic-vs:202405
    srl:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux

  links:
    - endpoints: ["sonic:eth1", "srl:e1-1"]

name: vm
topology:
  nodes:
    sonic:
      kind: sonic-vm
      image: vrnetlab/sonic_sonic-vs:202405

    sros:
      kind: nokia_srlinux
-     image: ghcr.io/nokia/srlinux
+     image: registry.wrkshpz.net/library/nokia_srl:24.7.1

  links:
    - endpoints: ["sonic:eth1", "srl:e1-1"]
```

Not only this gives us an easy way to share images with others, but also it enables stronger reproducibility of the lab, as the users of our lab would use exactly the same image that we built.
