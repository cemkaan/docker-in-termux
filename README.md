# Installing Docker in Termux
This repository contains instructions on how to install Docker in [Termux](https://termux.com/), a powerful terminal emulator for Android.

# Prerequisites
Before proceeding with the installation, make sure you have the following prerequisites:
- An Android device with Termux installed. You can download Termux from the [F-Droid](https://f-droid.org/packages/com.termux/) app store or github.
- Stable internet connection.
- Some easy way to write commands in terminal like remote access [SSH](https://wiki.termux.com/wiki/Remote_Access#Using_the_SSH_server)

# Installation Steps
Follow the steps below to install Docker in Termux:
1. Open Termux on your Android device.

2. Update and upgrade the packages by running the following command:
```bash
pkg update -y && pkg upgrade -y
```

3. Install the necessary dependencies by running the following command:
```bash
pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget -y
```
3.b. Run `termux-setup-storage` Check if you can access sdcard
```ash
~ $ df -h storage/external-1/
Filesystem      Size Used Avail Use% Mounted on
/dev/fuse       119G  27G   92G  23% /storage/8895-C850
```
go to sdcard
```bash
cd ~/storage/external-1/
```
4. Create a seperate directory:
```bash
mkdir alpine && cd alpine
```

5. Download Alpine Linux 3.19 (virt optimized) ISO:
```bash
wget https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/x86_64/alpine-virt-3.19.4-x86_64.iso
```

6. Create disk (note it won't actually take 45GB of space, more like 500-600MB):
```bash
qemu-img create -f qcow2 alpine.img 45G
```

7. Boot it up:
```bash
qemu-system-x86_64 -machine q35 -m 4000 -smp cpus=4 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -cdrom alpine-virt-3.19.4-x86_64.iso -nographic alpine.img
```

8. Login with username ``root`` (no password)

9. Setup network (press Enter to use defaults):
   localhost:~#
```bash
 setup-interfaces
```
 Available interfaces are: eth0.
 Enter '?' for help on bridges, bonding and vlans.
 Which one do you want to initialize? (or '?' or 'done') [eth0]
 Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp]
 Do you want to do any manual network configuration? [no]
 
localhost:~# 
```bash
ifup eth0
```

10. Create an answerfile to speed up installation:
```bash
wget https://gist.githubusercontent.com/cemkaan/a98d0c15d25ffa8d03da061a386b7879/raw/8759833ee329b2b8407461f5a6c7d7244d53a413/answerfile
```
> **NOTE:** You you see any error like this: ``wget: bad address 'gist.githubusercontent.com'``. Then run this command
> ```bash
> echo -e "nameserver 192.168.1.1\nnameserver 1.1.1.1" > /etc/resolv.conf
> ```

11. Patch ``setup-disk`` to enable serial console output on boot:
```bash
sed -i -E 's/(local kernel_opts)=.*/\1="console=ttyS0"/' /sbin/setup-disk
```

12. Run setup to install to disk
```bash
setup-alpine -f answerfile
```
say yes for disk install question
13. Once installation is complete, power off the VM (command ``poweroff``)

14. Boot again without cdrom:
```bash
qemu-system-x86_64 -machine q35 -m 3000 -smp cpus=4 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -nographic alpine.img

```

15. Update system and install docker:
```bash

echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf

apk update && apk add docker
```

16. Start docker:
```bash
service docker start
```

17. Enable docker on boot:
```bash
rc-update add docker
```

18. Check docker install successfully or not:
```bash
docker run hello-world
```
If it started hello-world container you can poweroff again. it's done.
now create a quick launcher 

# Quick launch
a- 
```bash
wget https://gist.githubusercontent.com/cemkaan/858961c9420037e2d00e0668563963d5/raw/dd4cd3b9a09bb8d88924299abc4c4e28a917a5e7/run_qemu.sh
```


b- chmod command: 
```bash
chmod +x run_qemu.sh
```
c-  first edit then run
```bash
./run_qemu.shbash run_qemu.sh
```
or `bash run_qemu.sh`
## Enable root access for SSH connections

```bash
apk add micro
micro /etc/ssh/sshd_config
```
Find the line PermitRootLogin prohibit-password and change it to:
**PermitRootLogin yes**
Save the changes and exit the editor. CTRL+s
Restart the SSH service:
```bash
service sshd restart
```
# Some useful keys
- ``Ctrl+a x``: quit emulation
- ``Ctrl+a h``: toggle QEMU console

# Usage
Now that Docker is installed in Termux, you can start using it to manage and run containers on your Android device. Refer to the official [Docker documentation](https://docs.docker.com/) for more information on how to use Docker.

# Contributing
If you encounter any issues during the installation process or have suggestions for improvements, please feel free to open an issue or submit a pull request.

# Acknowledgment
- This article inspired from: https://gist.github.com/oofnikj/e79aef095cd08756f7f26ed244355d62

# License
This project is licensed under the [MIT License](LICENSE).
