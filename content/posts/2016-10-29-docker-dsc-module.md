---
title: "PowerShell DSC Module for Docker - Initial Stab"
date: 2016-10-29T12:32:30-05:00
draft: false
tags: ["docker", "windows"]
---

So, first and foremost - this is a first take. I already have plans to add docker swarm provisioning, better system checks (e.g. verification that appropriate system patches are in place) and an option to select docker version (or latest) to install. 

<!--more-->

Nonetheless, I found it troubling that I couldnt find much in the way of installing Docker via DSC. So here we are. Notes: This is using Powershell Classes, so PS 5.0 is required; given that I dont think many people are deploying Docker in a scenario without 5.0, I'm not too worried about it :) So first and foremost; this is now in the PowerShell Gallery. [https://www.powershellgallery.com/packages/cDscDocker](https://www.powershellgallery.com/packages/cDscDocker) Update: also on GitHub: [https://github.com/walked/cDscDocker](https://github.com/walked/cDscDocker) And you can quickly install the Module with the following:



```
PS>  Install-Module -Name cDscDocker
```

Cool. Let's see about use:

```
Configuration dockerConfig
{
    Import-DscResource -module cDscDocker

    Node localhost{

		LocalConfigurationManager
		{
			RebootNodeIfNeeded = $true
		}

        cDscDocker ExampleA
        {
        docker = "docker"
        Ensure = 'Present'

        }
    }
}

dockerConfig
```

And finally, run DSC on the host, be it via push or pull.

```
#Only if you want to automatically reboot the node after docker install:
PS> Set-DscLocalConfigurationManager .\dockerConfig\ 

PS> Start-DscConfiguration .\dockerConfig\ -wait -force
```

Cool. What other featuures are planned for version 1.1:

*   Improved comments.
*   Expanded pre-flight-checks to cover necessary Server 2016 pre-Docker patching
*   Expended configuration via DSC
*   Swarm joins
*   Docker daemon exposure on external interfaces
*   Perhaps overlay network configurations?
*   More? We'll see!

  Full code for Version 1.0 _At an aside; it's really cool to see how Powershell 5.0 and PS Classes are incredibly powerful; doubly so if you have any experience with C#. Really phenomenal._

```powershell
enum Ensure
{
	Present
	Absent
}

[DscResource()]
class cDscDocker 
{
	[DscProperty(Key)] 
	[string]$docker

	[DscProperty(Mandatory)] 
	[Ensure]$Ensure

	#Equivalent of Set-TargetResource
	#Executes the current operation necessary to get the system to the desired state
	[void] Set()
	{
		if ($this.Ensure -eq [Ensure]::Present)
		{

			if (-not $this.CheckDockerProvider()) #If the docker provider isnt present
			{
				$this.InstallDockerProvider()

			}
			$this.InstallDocker() #Install docker..

		}

		elseif ($this.Ensure -eq [Ensure]::Absent)
		{
			Uninstall-Package -ProviderName DockerMsftProvider -Name docker -Force #Uninstall docker; will move to a function if re-used elsewhere
		}

	}

	#Equivalent of Test-TargetResource
	#Returns true if system is in the state specified; false if a modification needs to take place
	[bool] Test()
	{

		$dockerPresent = $this.CheckDocker()
		$returnValue = $false

		if ($this.Ensure -eq [Ensure]::Present)
		{

			if ($dockerPresent)
			{
				Write-Verbose "Item exists. Expected present"
				$returnValue = $true
			}
			if (-not $dockerPresent)
			{
				Write-Verbose "Item is Absent. Expect Present"
				$returnValue = $false
			}
		}
		Else
		{
			if ($dockerPresent)
			{
				Write-Verbose "Item exists. Expect Absent"
				$returnValue = $false
			}
			if (-not $dockerPresent)
			{
				Write-Verbose "Item absent. Expect Absent"
				$returnValue = $true
			}

		}

		return $returnValue

	}

	#Equivalent of the Get-TargetResource
	#Get's the current state of the system in cDscDocker object form
	[cDscDocker] Get()
	{
		if ($this.CheckDocker())
		{
			$this.Ensure = [Ensure]::Present
		}
		else
		{
			$this.Ensure = [Ensure]::Absent
		}

		return $this

	}

	#Function for installation of docker
	[void]InstallDocker()
	{
		Install-Package -Name docker -ProviderName DockerMsftProvider -Force
		$global:DSCMachineStatus = 1
	}

	#Function for installation of docker package provider
	[void]InstallDockerProvider()
	{
		Install-Module -Name DockerMsftProvider -Repository psgallery -Force
	}

	#Function to check if docker is already present
	[bool]CheckDocker()
	{
        $installed = $false

        if($this.CheckDockerProvider())
        {

		    $packages = Get-Package -ProviderName DockerMsftProvider
		    foreach ($p in $packages)
		    {
			    if ($p.Name -eq 'docker')
			    {
				    $installed = $true
			    }
		    }
        }

		return $installed
	}

	#Function to verify the appropriate docker package provider is available on the host
	[bool]CheckDockerProvider()
	{
		$available = $false
		$providers = Get-PackageProvider -ListAvailable

		foreach($provider in $providers)
        {
		    if ($provider.Name -eq "DockerMsftProvider")
		    {
			    $available = $true
		    }
        }

		return $available
	}
}


```

_Update: The version in the PS Gallery is now up to 1.0.5; please note this has some important fixes to the DSC Provider check, as well as implements automatic system reboots if your LCM settings support it._ 

_Update2: Code here is updated to reflect the important fixes. _