# Cmpe-283-Assignment2

Your assignment is to modify the CPUID emulation code in KVM to report back additional information when a
special CPUID “leaf function” is called.

- For CPUID leaf function %eax=0x4FFFFFFF:
  - Return the total number of exits (all types) in %eax
  - Return the high 32 bits of the total time spent processing all exits in %ebx
  - Return the low 32 bits of the total time spent processing all exits in %ecx
  - %ebx and %ecx return values are measured in processor cycles 

## Build Kernel
Before building the kernel, make sure to comment out these following 2 lines in the config file that you've copied over from the host: 
```
CONFIG_SYSTEM_TRUSTED_KEY
CONFIG_MODULE_SIG_KEY
```

Otherwise you will get this sort of error: 
```
make[1]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.
Makefile:1851: recipe for target 'certs' failed
make: *** [certs] Error 2

```

Proceed with the instructions below: 

```
sudo bash
apt-get install build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex libelf-dev
uname -a (and note down your kernel version)
cp /boot/config-$(uname -r) .config (make sure to comment out the debian certification information)
make oldconfig (use default options- continuously hold down enter)
make && make modules && make install && make modules_install
reboot
```

## Edit CPUID.C and VMX.C Files
Edit the CPUID and VMX Files with a leaf function for %eax=0x4FFFFFFF where information such as the number of exits and exit duration is returned. We printed this out to the kern.log file. After you've completed editting, rerun the make command from the previous step. If there are any syntax errors in your c files, it will be flagged when building the kernel.


## Install Virt-Manager
Next you will need to install virt-manager to install a nested VM inside your current VM. Make sure hardware virtualization is also enabled for the outer VM. 
```
sudo apt-get install virt-manager 
```
Follow these [instructions](https://www.tecmint.com/create-virtual-machines-in-kvm-using-virt-manager/) to create a VM via *virt-manager*. You will need to download an Ubuntu ISO image (in our case we used 20.04).

## Test CPUID.C & VMX.C Changes
Use Ubuntu's CPUID package to force an exit for 0x4FFFFFFF. 
```
sudo apt-get update 
sudo apt-get install cpuid

# run CPUID instruction
cpuid -l 0x4FFFFFFF
```
You can tail the kern.log file from the host VM so that you can see the output each time you execute CPUID. 
```
tail -f /var/log/kern.log
```

## Helpful Links
[Check if Nested Virtualization is Enabled](https://ostechnix.com/how-to-enable-nested-virtualization-in-kvm-in-linux/)
[Build Simple Linux Kernel Module](https://www.geeksforgeeks.org/linux-kernel-module-programming-hello-world-program/)

## Followup Questions
1. Linda and Dhruwaksh researched each step together, so we could both understand the process. We would also encounter similar errors, so we both research them, then communicate our findings afterwards to resolve them. For file changes, Linda editted the CPUID.C and Dhruwaksh editted the VMX.C based on the CPUID file. 

2. Steps to complete assignment are described above. 

3. Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there
more exits performed during certain VM operations? Approximately how many exits does a full VM
boot entail?

  Yes, number of exits increase. At boot, the exits were around 1~1.3 million. Different tasks result in increasing the total number of exits. The rate doesn't seem stable initially, but after that it depends on the task. I/O operations and similar tasks also result in increasing number of exits.
