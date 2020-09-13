# Kubernetes on Raspberry Pi

[Build Reference](https://wiki.learnlinux.tv/index.php/Setting_up_a_Raspberry_Pi_Kubernetes_Cluster_with_Ubuntu_20.04)

## Set-up Process (do the following on each Raspberry Pi)

* [Download and flash Pi SD card](https://ubuntu.com/download/raspberry-pi)
* [Download Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/files/latest/download?source=navbar)

* Flash SD Card

![Flash SSD card](/images/win32diskimager.png)

```
use “ubuntu” for the username and the password. You will be asked to change this default password after you log in.
```

* Edit the host name

```
Edit /etc/hosts and /etc/hostname on the SD card to the actual name of the instance

For example:

 k8s-master

 k8s-worker-01

(Or whatever naming scheme you wish)
```

* Configure boot options
```
Edit /boot/firmware/cmdline.txt and add:

cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1

Note: Add that to the end of the first line, do not create a new line.
```

* Install all updates
```
 sudo apt update && sudo apt dist-upgrade
 ```
 