# As of October 2016

Running both Windows and Linux containers on Windows 10 is easier than ever!

## Install Docker

### For Windows 10: Install Docker For Windows **Beta**

If you install the current _stable_ version of Docker For Windows, you'll only be able to run Linux images.  Either way, the full installer for Docker for Windows will take care of enabling the (Hyper-V and) Containers feature (which may require a reboot).  But the _beta_ version will include both the Windows and Linux daemons.  [Direct installer link](https://download.docker.com/win/beta/InstallDocker.msi).

### For Windows Server 2016: Install the Docker Provider

Note: I haven't gotten Linux VMs running doing it this way yet...

For this one, you can [follow Microsoft's walk-through](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server). The Windows 10 guide is overly complicated, but on Server 2016 it's really simple, just run a couple of PowerShell commands and (if you didn't have Hyper-V installed previously) reboot.

```posh
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

## Install the Docker module?

You don't need the [docker module](https://github.com/Microsoft/Docker-PowerShell), it merely provides an alternative to the `docker` client, and its command-line interface.  The nice thing about it is that it can manage Docker locally or remotely on both Windows and Linux, and does **note** require the docker client. However, it currently can't manage "compose" or "swarm" mode, so you'll likely still need the docker command-line executable.

Since the module is still in **alpha** status, it's not on the main PowerShell gallery, you need to add their appveyor feed to your system:

```posh
Register-PSRepository -Name DockerPS-Dev -SourceLocation https://ci.appveyor.com/nuget/docker-powershell-dev
Install-Module Docker -Repository DockerPS-Dev -Force
```

By default, Docker For Windows installs a little bridge (with a systray icon) to let non-elevated shells run `docker` commands, and to help start the services. It's a neat idea, but it's complicated by the fact that there are _currently_ two services it has to talk to (Windows and Linux hosts). 

To talk to the Windows engine, you right-click the systray icon and choose "Switch to Windows containers..." then you can run commands and manage nano and server core images and containers! Of course, if you want to manage a Linux container, you have to right-click on the systray again and switch back (or run a simple command-line `& 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon`).

For more information about how that system works [check out stefan Scherer's post](https://stefanscherer.github.io/run-linux-and-windows-containers-on-windows-10/)

If you really need to manage both at once, or you just have a hankering for doing things the hard way, you can get a little creative with me.  The two services (windows and linux) each listen on a named pipe, so you can actually manage either of them (from an elevated command line), and you even can set up a PowerShell function to run `docker.exe` with the right parameter, or a hashtable that you can splat to the Docker module commands.

```posh
function dw { docker -H npipe:////./pipe/docker_engine_windows @args }
function dl { docker -H npipe:////./pipe/docker_engine @args }

$dw = @{ HostAddress = 'npipe://./pipe/docker_engine_windows' }
$dl = @{ HostAddress = 'npipe://./pipe/docker_engine' }
```

That will let you run commands like:

```posh
> Get-ContainerImage @dl

RepoTags                                 ID                   Created                        Size(MB) 
--------                                 --                   -------                        -------- 
ubuntu:latest                            sha256:f753707788... 10/13/2016 9:13:21 PM          121.27   

> Get-ContainerImage @dw

RepoTags                                 ID                   Created                        Size(MB)
--------                                 --                   -------                        --------
microsoft/nanoserver:latest              sha256:3a703c6e97... 6/16/2016 3:57:35 PM           924.92  
```

Or without the module, run docker via the functions:

```posh
> dw images                                                                                                                                                                      
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE                                                                                                               
microsoft/nanoserver   latest              3a703c6e97a2        4 months ago        969.8 MB                                                                                  

> dl images                                                                                                                                                                      
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE                                                                                                                  
ubuntu              latest              f753707788c5        2 weeks ago         127.2 MB   
```
