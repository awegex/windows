# Ubuntu
Enable WSL support as **administrator**.

```cmd
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Reboot Windows.

```cmd
shutdown /r /t 0
```

Install [WSL 2 Linux Kernel](https://aka.ms/wsl2kernel), then configure WSL.

```cmd
wsl --set-default-version 2
```

Install and launch [Ubuntu](https://aka.ms/wslstore).

```sh
sudo visudo -c

sudo EDITOR=tee visudo >/dev/null <<'EOF'
# Locale settings.
Defaults env_keep += "LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET"

# Profile settings.
Defaults env_keep += "MM_CHARSET EDITOR PAGER LS_COLORS TMUX SESSION USERPROFILE"

# User privilege specification.
root  ALL=(ALL) ALL
%sudo ALL=(ALL) NOPASSWD: ALL

# See sudoers(5) for more information on "#include" directives:
#includedir /etc/sudoers.d
EOF

exit
```

Set Ubuntu as the default WSL distribution and start it.

```sh
wsl -s Ubuntu
wsl -d Ubuntu
```

Update system.

```sh
sudo apt update
sudo apt upgrade -y
sudo apt autoremove --purge -y
sudo apt clean
```

Remove snapd.

```sh
sudo apt purge snapd -y
sudo apt autoremove --purge -y
sudo rm -rf /root/snap /snap
sudo apt clean
```

Install packages.

```sh
sudo apt install -y ccze net-tools p7zip pv pwgen tree wipe zip
sudo apt install -y -o APT::Install-Suggests=0 -o APT::Install-Recommends=0 pngcrush
```

Configure system.

```sh
sudo curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/tmux.conf -o /etc/tmux.conf
sudo curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/bash.sh -o /etc/profile.d/bash.sh
sudo curl -L https://raw.githubusercontent.com/qis/windows/master/wsl/wsl.sh -o /etc/profile.d/wsl.sh
sudo chmod 0755 /etc/profile.d/bash.sh /etc/profile.d/wsl.sh
```

Configure WSL.

```sh
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[automount]
enabled=true
options=case=off,metadata,uid=1000,gid=1000,umask=022
EOF
```

Disable message of the day.

```sh
sudo sed -E 's/^(session.*pam_motd\.so.*)/#\1/' -i /etc/pam.d/*
```

Replace shell config files.

```sh
sudo rm -f /{root,home/*}/.{bashrc,profile,viminfo}
ln -s /etc/profile.d/bash.sh ~/.bashrc
```

Exit shell to release `~/.bash_history`.

```sh
exit
```

Restart distribution to apply `/etc/wsl.conf` settings.

```cmd
wsl -t Ubuntu
wsl -d Ubuntu
```

Configure `vim`.

```sh
sudo rm -rf /etc/vim
sudo git clone https://github.com/qis/vim /etc/vim
sudo touch /root/.viminfo
touch ~/.viminfo
```

Clean home directory files.

```sh
sudo rm -f /root/.bash_history /root/.bash_logout
rm -f ~/.bash_history ~/.bash_logout
sudo touch /root/.hushlogin
touch ~/.hushlogin
```

Create **user** home directory symlinks in WSL.

```sh
mkdir -p ~/.ssh; chmod 0700 ~/.ssh
for i in authorized_keys config id_rsa id_rsa.pub known_hosts; do
  ln -s "${USERPROFILE}/.ssh/$i" ~/.ssh/$i
done
sudo chown `id -un`:`id -gn` "${USERPROFILE}/.ssh"/* ~/.ssh/*
sudo chmod 0600 "${USERPROFILE}/.ssh"/* ~/.ssh/*
ln -s "${USERPROFILE}/.gitconfig" ~/.gitconfig
```

## Development
Install basic development packages.

```sh
sudo apt install -y binutils-dev debconf-utils libc6-dev libgcc-9-dev manpages-dev
sudo apt install -y -o APT::Install-Suggests=0 -o APT::Install-Recommends=0 \
  autoconf automake bison flex gdb make nasm ninja-build pkgconf sqlite3
```

Install [CMake](https://cmake.org/).

```sh
sudo rm -rf /opt/cmake; sudo mkdir -p /opt/cmake
wget https://github.com/Kitware/CMake/releases/download/v3.18.4/cmake-3.18.4-Linux-x86_64.tar.gz
sudo tar xf cmake-3.18.4-Linux-x86_64.tar.gz -C /opt/cmake --strip-components=1
rm -f cmake-3.18.4-Linux-x86_64.tar.gz
sudo tee /etc/profile.d/cmake.sh >/dev/null <<'EOF'
export PATH="/opt/cmake/bin:${PATH}"
EOF
sudo chmod 0755 /etc/profile.d/cmake.sh
. /etc/profile.d/cmake.sh
```

Install [Node](https://nodejs.org/).

```sh
sudo rm -rf /opt/node; sudo mkdir -p /opt/node
wget https://nodejs.org/dist/v12.16.3/node-v12.16.3-linux-x64.tar.xz
sudo tar xf node-v12.16.3-linux-x64.tar.xz -C /opt/node --strip-components=1
rm -f node-v12.16.3-linux-x64.tar.xz
sudo tee /etc/profile.d/node.sh >/dev/null <<'EOF'
export PATH="/opt/node/bin:${PATH}"
EOF
sudo chmod 0755 /etc/profile.d/node.sh
. /etc/profile.d/node.sh
```

Install [LLVM](https://llvm.org/).

```sh
sudo rm -rf /opt/llvm; sudo mkdir -p /opt/llvm
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-11.0.0/clang+llvm-11.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz
sudo tar xf clang+llvm-11.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz -C /opt/llvm --strip-components=1
rm -f clang+llvm-11.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz
sudo tee /etc/profile.d/llvm.sh >/dev/null <<'EOF'
export PATH="/opt/llvm/bin:${PATH}"
EOF
sudo chmod 0755 /etc/profile.d/llvm.sh
. /etc/profile.d/llvm.sh
```

Set default system compiler.

```sh
sudo update-alternatives --remove-all cc
sudo update-alternatives --remove-all c++
sudo update-alternatives --install /usr/bin/cc  cc  /opt/llvm/bin/clang   100
sudo update-alternatives --install /usr/bin/c++ c++ /opt/llvm/bin/clang++ 100
```

<!--
clang++ -std=c++20 -stdlib=libc++ -Os -flto=full main.cpp -fuse-ld=lld
clang++ -std=c++20 -stdlib=libc++ -Os -flto=full main.cpp -fuse-ld=lld -static-libstdc++ /opt/llvm/lib/libc++abi.a
-->
