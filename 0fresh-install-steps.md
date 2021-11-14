# TLDR

- fucked up my lubuntu to oblivion
- should have written this years ago

## TODO

- automate this shiz like the guy did his with ansible
- [checkout spack](https://spack-tutorial.readthedocs.io/en/latest/)
- [checkout nix](https://nixos.org/)

## links

- [initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
- [point domain name to your servers ip](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)
- [configure ssh key based authentication and disable password authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
- [common ufw firewall rules](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
- [upgrading bash on mac](https://itnext.io/upgrading-bash-on-macos-7138bd1066ba)

## fresh install

### dev

1. download & login to firefox dev edition
2. setup github
   - [github cli](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)
   - [verify your commits](https://docs.github.com/en/authentication/managing-commit-signature-verification)
3. setup vscode
   - [vscode settings](https://gist.github.com/noahehall/33f60c724f51bde9afa2c2a9e540d094)

#### ubuntu:dev

1. setup ubuntu
   - [fk snaps, disable that shiz](https://www.simplified.guide/ubuntu/remove-snapd)
   - dconf-editor
   - shutter
   - synaptic package manager
   - bleachbit
   - gnome extensions
   - apt-updatoer
   - neofetch
   - system-monitor
   - apt-transport-https
   - [allota shit from here](http://packages.azlux.fr/)
   - setup bash
   - damn my skillz is fallin off
   - rclone
   - oha
   - gping
   - duf
   - dockly
   - bpytop
   - htpo
   - grv
   - setup dev
   - docker

5. [setup chromium](https://linuxize.com/post/how-to-install-chromium-web-browser-on-ubuntu-20-04/)

#### mac:dev

- xcode install
- $ touch ~/.bashrc
- $ echo ". ~/.bashrc" > ~/.bash_profile
- upgrade bash (apple refuses to update pass bash v3.2) cuz fk apple
- setup bash as defualt shell (FCK ZSH)
- switch to iterm2
- [macports](https://www.scrim.psu.edu/support/userspace-macports.html)
- continue with ubuntu setup

### remote server setup

  1. setup non root user with sudo priv
  2. configure ssh key-based authentication
  3. disable password authentication and enable ufw with ssh enabled
  4. setup firewall (usually ufw) to block everything except 80, 443, and 22 initially and define your default policies