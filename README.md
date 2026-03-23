# Stremio for Fedora — Build Guide

Stremio does not provide an official `.rpm` package for Fedora. This guide documents how to build one yourself using Docker as a build environment. The result is a **native `.rpm`** that installs and runs like any normal Fedora package — no sandboxing, no Flatpak.

Tested on **Fedora 43**.

---

## Prerequisites

- Git
- Docker

Install them if you haven't already:

```bash
sudo dnf install git docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

---

## Steps

### 1. Clone the Stremio shell repo

```bash
cd ~
git clone --recurse-submodules https://github.com/Stremio/stremio-shell.git
cd stremio-shell
```

> `--recurse-submodules` is required — the build will fail without it.

### 2. Download the Fedora build files

Grab the latest release tarball from the [Stremio shell releases](https://github.com/Stremio/stremio-shell/releases) page, extract it, and copy the Fedora distro folder into your cloned repo:

```bash
cp -r ~/stremio-shell-*/distros/Fedora ~/stremio-shell/distros/fedora
cd ~/stremio-shell/distros/fedora
```

### 3. Fix a broken package name in the Dockerfile

The Dockerfile references `qt5-devel` which no longer exists in newer Fedora. Open `Dockerfile` and replace:

```
qt5-devel
```

with:

```
qt5-qtbase-devel
```

### 4. Generate the spec file

```bash
bash mkconfig.sh
```

### 5. Fix the same broken package name in the spec file

Open `stremio.spec` and replace:

```
BuildRequires: qt5-devel
```

with:

```
BuildRequires: qt5-qtbase-devel
```

### 6. Build the Docker image

```bash
docker build -t stremio-fedora .
```

This will take a few minutes the first time as it pulls the Fedora image and installs all build dependencies.

### 7. Build the RPM

```bash
docker run --name stremio-build stremio-fedora bash -c "cd /home/builduser && bash package.sh"
```

This will take 10–20 minutes depending on your machine.

### 8. Copy the RPM out of the container

```bash
docker cp stremio-build:/root/rpmbuild/RPMS/x86_64/stremio-*.rpm .
docker rm stremio-build
```

### 9. Install

```bash
sudo dnf install ./stremio-*.rpm
```

### 10. Clean up Docker (optional)

```bash
docker rmi stremio-fedora
docker system prune
```

---

## Notes

- The OpenPGP warning during install is expected for locally built packages and is harmless.
- The symlink warning during the RPM build (`absolute symlink: /opt/stremio/node`) is also harmless.
- Docker is only used as a **build environment**. The installed app runs fully natively with no containerization.
- To build a specific branch, edit `package.sh` and pass the branch name, or run: `bash package.sh your-branch-name`

---

## Credits

- [Stremio](https://www.stremio.com) — all credit for the actual software goes to the Stremio team. This repo is just a build guide.
- Built from the official [stremio-shell](https://github.com/Stremio/stremio-shell) source, which is MIT licensed.
