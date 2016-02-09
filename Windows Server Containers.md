Windows Server Containers
==================

### Manually Create VM image

Mount the Server 2016 iso and go to the NanoServer directory in the root.

```powershell
Import-Module .\NanoServerImageGenerator.psm1

$adminPassword = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force

New-NanoServerImage
    -MediaPath E:\
    -BasePath D:\Nano\Base
    -TargetPath D:\Nano\Image\Nano.vhd
    -ComputerName Nano
    -AdministratorPassword $adminPassword
    -OEMDrivers
    -ReverseForwarders
    -Containers
```

Once the VHD is created you can create a new VM with it using VirtualBox or Hyper-V

### Vagrant

http://www.hurryupandwait.io/blog/a-packer-template-for-windows-nano-server-weighing-300mb

Build vagrant box image using packer template:
https://github.com/mwrock/packer-templates

```powershell
vagrant init mwrock/WindowsNano
vagrant up --provider virtualbox
```

### Administration

Create a local account on nano server:
```powershell
net user david /add *
net localgroup administrators david /add
```

To connect to a file share on the nano server:
```powershell
net use \\192.168.1.154\c$ /user:david P@ssw0rd
```

### Containers
```powershell
Find-PackageProvider
Install-PackageProvider ContainerProvider -Force
Find-ContainerImage
Install-ContainerImage NanoServer
```

### Create a virtual switch
```powershell
New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```

### Start the docker daemon
```powershell
$cred = Get-Credential david
Enter-PSSession 192.168.1.154 -Credential $cred
start-process cmd "/k docker daemon -D -b `"Virtual Switch`" -H 0.0.0.0:2375"
```

### Firewall

```
https://github.com/Microsoft/DockerTools/blob/master/ConfigureWindowsDockerHost.ps1
```

Just disable it
```powershell
netsh advfirewall set domainprofile state off
netsh advfirewall set privateprofile state off
netsh advfirewall set publicprofile state off
netsh advfirewall set currentprofile state off
```

```powershell
dnu publish . --no-source --native --runtime C:\Users\david\.dnx\runtimes\dnx-coreclr-win-x64.1.0.0
-rc1-update1
```

### Configure Hosting Environment

```
SETX ASPNET_ENV Development
```
