---
title: "Jenkins, Powershell DSC, and CI"
date: 2017-01-11T12:32:30-05:00
draft: false
tags: ["windows", "powershell"]
---

DSC is truly one of my favorite things to come out of the PowerShell team to date. The power for idempotent infrastructure and deployment is great. However, one _relatively_ minor roadblock is getting an MOF delivery pipeline in place for getting the MOF configurations to a pull server.

<!--more-->

 I'm fairly happy with a very basic Jenkins implementation to manage this. Certainly beats manually compiled mof files.  
 
 Let's run through what this pipeline looks like:

1. Git is utilized for a repository for all DSC configuration files; **but not mof files** (although I guess it's reasonable that this could include mof files as they are just text files; I'm considering the mof a build artifact and thus not included in the git repo)
2. Jenkins will maintain a sync with the git repository, triggering a build when a change is detected
3. Jenkins will compile the mof files as a build action
4. Optional: you can run / validate against pester tests; not covered in this post
5. Jenkins will publish to an SMB share with the build artifacts, renamed for easy integration with DSC Configuration Names (**note: we're using configuration names not configuration IDs!)**
6. The DSC pull server can either be triggered to sync with the target directory, or simply have it's own sync mechanism. This isnt covered in this post as it's trivial to implement and highly dependent upon your environment.


_Sidenote: I may do a video on this environment configuration a bit later as this is something that strikes me as a "better seen than read"._ 


So let's run through this at lightspeed 
![ftl]({{site.url}}/assets/images/misc/ftl.gif)  

### Step 1: Git is configured as a repository for your DSC configuration files

For this example, I'm using a private GitLab repository, but you can use whatever you want. Inside the repository, I have a simple directory structure

```
Root Directory
└─ DSC
    ├──── Configurations
    ├──── Examples
    └──── Scripts
```

Inside the configurations directory, I have a bunch of configuration files, one example is (simplified) below:

```powershell
Configuration webservice
{
    param ($MachineName)
        Node $MachineName
        {
                #Install IIS Role
                WindowsFeature IIS
                {
                    Ensure = "Present"
                    Name = "Web-Server"
                }
                #Install ASP.NET 4.5
                WindowsFeature ASP
                {
                    Ensure = "Present"
                    Name = "web-Asp-Net45"
                }
        }
}

webservice -MachineName localhost
```

Cool.

### Step 2: Jenkins is configured to poll that git repository for changes.

![jenkins1]({{site.url}}/assets/images/dsc-jenkins/jenkins-1.jpg) 

Pretty self explanatory.

### Step 3: Jenkins will compile the mof files as a build action

_This requires the Jenkins PowerShell plugin to execute PowerShell._

```powershell
Set-location $env:workspace\DSC\Configurations
$scripts = Get-ChildItem $env:workspace\DSC\Configurations -recurse -include "*.ps1"

foreach($script in $scripts){
 & $script
}
$mofFiles= Get-ChildItem $env:workspace\DSC\Configurations -recurse -include "*.mof"
foreach($mof in $mofFiles){
    Rename-Item $mof "$($mof.directory.Name).mof"
    New-DscChecksum "$($mof.DirectoryName)\$($mof.Directory.Name).mof"
}
```

So what does this do? 

Mainly two things: 

1. It executes every ps1 file in the Configurations directory (based on the directory structure referenced above) 
2. It renames all the compiled mof to be the same as directoryname.mof - as we're using named configurations in deployment, we want to name these with the configuration name we'll be referencing on the clients. 2.5) Also generating the DdsChecksum here, why not get that done?

### Step 5: Publication to an SMB Share from Jenkins

_Yes I know I skipped 4; that's because I left out pester tests for this particular blog post._ This, once again, is pretty simple. 

Simply use the CIFS Jenkins plugin. 

![jenkins2]({{site.url}}/assets/images/dsc-jenkins/jenkins-2.jpg)  

### Step 6: Sync your DSC pull server Configuration directory with the SMB Share

** This is really the simplest step. Just use your imagination. 

![imagination]({{site.url}}/assets/images/misc/imagination.gif)   

Really, the trickiest part is the PowerShell script for Jenkins to build/process the mof files. But the value in having CI setup for your DSC configurations is super duper awesome. No more manual copying.