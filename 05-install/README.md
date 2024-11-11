# Containerlab Installation

## Docker

Install docker using the handy script:

```bash
curl -L http://containerlab.dev/setup \
| sudo bash -s "install-docker"
```

Add yourself to the `docker` group to enable sudo-less docker commands:

```bash
sudo usermod -aG docker $USER
```

Now log out and log in for the group change to take effect.

Check that docker is installed and running:

```bash
docker run --rm hello-world
```

Expected output: Hello from Docker!

Alternative installation options can be found [here](https://docs.docker.com/engine/install/).

## Containerlab

Install containerlab using the installation script:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

> If you hit a github rate limit error, you can download container from a package repository following the instruction provided in the [installation guide](https://containerlab.dev/install/#package-managers).

Alternative installation options are available [here](https://containerlab.dev/install/).

## One command to install Containerlab and Docker

***For reference only***
In place of the above 2 commands to install Docker and Containerlab separately, there is a single script that will install both components along with GitHub CLI in one go.

```bash
curl -sL https://containerlab.dev/setup | sudo -E bash -s "all"
```

```bash
docker run --rm hello-world
```

Expected output: Hello from Docker!
