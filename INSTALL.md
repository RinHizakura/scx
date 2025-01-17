# Install Instructions by Distro

## Ubuntu

Experimental sched_ext support for Ubuntu is provided by the following
launchpad project:

 https://launchpad.net/~arighi/+archive/ubuntu/sched-ext

#### Upgrading to 24.04 (NobleNumbat)

Currently, only the 24.04 release is supported. You can upgrade to 24.04
using the following command:

```
$ sudo do-release-upgrade -d
```

#### Installing the Kernel and Schedulers

```
$ sudo add-apt-repository -y --enable-source ppa:arighi/sched-ext
$ sudo apt install -y linux-generic-wip scx
$ sudo reboot
```

After the reboot, the scheduler binaries in `/usr/sbin/scx_*` should be usable.
Note: they must be called with `sudo` like other BPF programs e.g. `sudo scx_simple`.

#### Setting up Dev Environment

```
$ apt source scx
$ sudo apt build-dep scx
```

## Arch Linux

Import the gpg key. This can be skipped if the signature checking is disabled.

```
$ sudo pacman-key --recv-keys F3B607488DB35A47 --keyserver keyserver.ubuntu.com
$ sudo pacman-key --lsign-key F3B607488DB35A47
```

If you haven't imported the GPG key, append the following line.

```
SigLevel = Never
```

#### Adding the Repository

Install packages with a list of mirrors and GPG keys

```
$ sudo pacman -U 'https://mirror.cachyos.org/repo/x86_64/cachyos/cachyos-keyring-20240331-1-any.pkg.tar.zst' 'https://mirror.cachyos.org/repo/x86_64/cachyos/cachyos-mirrorlist-18-1-any.pkg.tar.zst'
```

Add the following custom repository section to `/etc/pacman.conf`.

```
# cachyos repos
[cachyos]
Include = /etc/pacman.d/cachyos-mirrorlist
```

#### Installing the Kernel and Schedulers

```
$ sudo pacman -Sy cachyos/linux-sched-ext cachyos/linux-sched-ext-headers cachyos/scx-scheds
```

:warning: The kernel installs as `/boot/vmlinuz-linux-sched-ext` along with
the matching initramfs. Update the bootloader configuration to add the boot
entry for the new kernel.

#### Setting Up Dev Environment

In addition to the packages from the previous step, install the following.

```
$ sudo pacman -Sy meson cargo bpf pahole
```

#### Using Debug Kernel

CachyOS does provide a kernel with an unstripped vmlinux, which can be used for debugging.

```
$ sudo pacman -Sy linux-cachyos-sched-ext-debug linux-cachyos-sched-ext-debug-headers
```

## Fedora

Experimental sched_ext support for Fedora is provided by the following
copr project:

 https://copr.fedorainfracloud.org/coprs/jordalgo/fedora-6.8-sched_ext/

#### Installing the Kernel

```
$ sudo dnf copr enable jordalgo/fedora-6.8-sched_ext
$ sudo dnf --disablerepo="*" --enablerepo="copr:copr.fedorainfracloud.org:jordalgo:fedora-6.8-sched_ext" upgrade kernel
$ sudo reboot
```

Then select the kernel containing the name "schedext".

#### Installing the Schedulers

At the moment, only the Rust schedulers are available for install (c schedulers coming soon):
- scx_rusty
- scx_layered
- scx_rustland

Example:
```
$ sudo dnf install scx_rusty
$ sudo scx_rusty
```

#### Setting up Dev Environment

No additional steps needed here other than what is mentioned in the main README.md.

## Nix

(Chaotic Nyx)[https://github.com/chaotic-cx/nyx] is maintaining the linux-cachyos kernel and scx-scheds package in a flake.

#### Integrate the repository using flake

<pre lang="nix"><code class="language-nix">
{
  description = "My configuration";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    chaotic.url = "github:chaotic-cx/nyx/nyxpkgs-unstable";
  };

  outputs = { nixpkgs, chaotic, ... }: {
    nixosConfigurations = {
      hostname = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          ./configuration.nix # Your system configuration.
          chaotic.nixosModules.default # OUR DEFAULT MODULE
        ];
      };
    };
  };
}
</code></pre>

#### Add this to your configuration to install the kernel

<pre lang="nix"><code class="language-nix">
{
  boot.kernelPackages = pkgs.linuxPackages_cachyos;
  environment.systemPackages =  [ pkgs.scx ];
}
</code></pre>

Then install the package and reboot your system. After you can use all provided example schedulers.
