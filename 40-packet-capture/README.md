# Packet capture

Every lab emulation software must provide its users with the packet capturing abilities. Looking at the frames as they traverse the network links is not only educational, but also helps to troubleshoot the issues that might arise during the lab development.

Containerlab offers a simple way to capture the packets from any interface of any node in the lab since every interface is exposed to the underlying Linux OS. This article will explain how to do that.

Everything we are going to do in this exercise is explained in details in the [Containerlab documentation](https://containerlab.dev/manual/wireshark/).

## Local capture

To capture the packets on the local host VM directly, run the below command on the VM host. This captures packets on the SR Linux interface `ethernet-1/1 `.

```bash
sudo ip netns exec clab-vm-srl tcpdump -nni e1-1
```

The output is displayed directly on the screen after the capture is stopped using CTRL+c.

```bash
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on e1-1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C17:17:45.763237 IP 10.0.0.0 > 10.0.0.1: ICMP echo request, id 43538, seq 8, length 64
17:17:45.764509 IP 10.0.0.1 > 10.0.0.0: ICMP echo reply, id 43538, seq 8, length 64
17:17:45.953099 LLDP, length 243: sonic
17:17:46.764454 IP 10.0.0.0 > 10.0.0.1: ICMP echo request, id 43538, seq 9, length 64
17:17:46.765547 IP 10.0.0.1 > 10.0.0.0: ICMP echo reply, id 43538, seq 9, length 64
17:17:47.766626 IP 10.0.0.0 > 10.0.0.1: ICMP echo request, id 43538, seq 10, length 64
17:17:47.767523 IP 10.0.0.1 > 10.0.0.0: ICMP echo reply, id 43538, seq 10, length 64

7 packets captured
7 packets received by filter
0 packets dropped by kernel
```


## Remote capture

To capture the packets from a containerlab node running on a remote host that we are going to explore is called "remote capture". In this scenario a user has a network connectivity (ssh) to the host that runs containerlab topology and wishes to get the packet capture displayed in the Wireshark running locally.

To achieve this, we will execute the `tcpdump` command on the remote host and pipe the output to the local Wireshark app. Here is a command that does it all.

It captures the traffic from SR Linux (`clab-vm-srl`) port `ethernet-1/1` running on your host and displaying the capture in the Wireshark.

Login to `clab-vm-srl` and initiate a ping to `10.0.0.0`:

```srl
ping 10.0.0.0 network-instance default
```

<small>The command is provided for WSL and Mac systems, assuming default Wireshark installation path. Replace `X` with your VM number.</small>

Windows/WSL:

```bash
ssh autoconuser@<X>.wrkshpz.net \
"sudo ip netns exec clab-vm-sros tcpdump -U -nni eth1 -w -" | \
/mnt/c/Program\ Files/Wireshark/wireshark.exe -k -i -
```

macOS:

```bash
ssh autoconuser@<X>.wrkshpz.net \
"sudo ip netns exec clab-vm-sros tcpdump -U -nni eth1 -w -" | \
/Applications/Wireshark.app/Contents/MacOS/Wireshark  -k -i -
```

## Edgeshark

[Edgeshark](https://edgeshark.siemens.io/#/) is a set of tools that offer (among many things) a Web UI that displays every interface of every container and can start a wireshark as easy as clicking a button.

Edgeshark installation consists of two parts:

1. A service that runs on the host that runs containerlab topologies
2. A wireshark capture plugin that runs next to the Wireshark on a user's PC

To install the service, paste the installer command that uses docker compose to deploy the service:

```bash
curl -sL \
https://github.com/siemens/edgeshark/raw/main/deployments/wget/docker-compose.yaml | \
docker compose -f - up -d
```

Now, you have to install the client plugin based on the OS of your PC.

### Windows

Windows users get to enjoy a simple installer-based workflow that installs the URL handler and the Wireshark plugin in one go.

Download the installer file - <https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_windows_amd64.zip>

Unzip the archive and launch the installer script.

### MacOs

MacOs users have to suffer a little. But it is not that bad either.

To install the URL handler paste the following in the Mac terminal app:

```bash
mkdir -p /tmp/pflix-handler && cd /tmp/pflix-handler && \
rm -rf packetflix-handler.zip packetflix-handler.app __MACOSX && \
curl -sLO https://github.com/srl-labs/containerlab/files/14278951/packetflix-handler.zip && \
unzip packetflix-handler.zip && \
sudo mv packetflix-handler.app /Applications
```

To install the extpcap wireshark plugin execute in the Mac terminal:

```bash
# for x86_64 MacOS use https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_amd64.tar.gz
DOWNLOAD_URL=https://github.com/siemens/cshargextcap/releases/download/v0.10.7/cshargextcap_0.10.7_darwin_arm64.tar.gz
mkdir -p /tmp/pflix-handler && curl -sL $DOWNLOAD_URL | tar -xz -C /tmp/pflix-handler && \
open /tmp/pflix-handler && open /Applications/Wireshark.app/Contents/MacOS/extcap
```

The command above will open two Finder windows, one with the `cshargextcap` binary and the other with the Wireshark's existing plugins. Move the `cshargextcap` file over to the window with Wireshark plugins.

### Web UI

To access the Edgeshark UI, open a browser and navigate to the following URL (substitute the hostname with your assigned VM):

<http://{ID}.wrkshpz.net:5001>

Note, the http schema is important, since https is not enabled.
