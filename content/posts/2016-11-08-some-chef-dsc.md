---
title: "Some Chef, Some DSC - Website Deployment"
date: 2016-11-02T12:32:30-05:00
draft: false
tags: ["chef", "windows"]
---

If you know me, you know I like what DSC does. What recently came to my attention is that Chef can leverage DSC and that's pretty cool.

<!--more-->

 _First and foremost; the disclaimer: I'm not particularly well versed in Chef; there's probably better ways to do this. I know there's an IIS cookbook in the Chef supermarket; but for testing I had a few troubles with this running the chef client in local mode._ 
 
 Now; let's look at some DSC inside of Chef.

```ruby
dsc_script 'Web-Server' do
    code <<-EOH
    WindowsFeature InstallWebServer
    {
        Name = "Web-Server"
        Ensure = "Present"
    }

    EOH
end
```

Really pretty simple; it allows us to use the idempotent nature of native DSC without having to build our checks into the actual usage like this example from the Chef tutorial:

```ruby
powershell_script 'Install IIS' do
  code 'Add-WindowsFeature Web-Server'
  guard_interpreter :powershell_script
  not_if "(Get-WindowsFeature -Name Web-Server).Installed"
end
```

Why does it matter? It may or may not matter that much depending on the resources you're using, but to me being able to leverage DSC inside of Chef, without building out a full DSC architecture seems so elegant to me. Chef handles the MOF compilation and the LCM configuration to actually execute the DSC configuration. Let's look at a full Chef script that'll put together a baseline web server, hit a remote repository for a zip file, and extract it into the IIS directory - only when changed.   First off, there's a pair of actions to draw attention to:

```ruby
remote_file "Code Source" do
    source "file:////synology.home.lan//storage//deploy//TestApplication.zip"
    path "C:/temp/TestApplication.zip"
    notifies :run, 'powershell_script[Extract Archive]', :immediately
end

powershell_script 'Extract Archive' do
    code 'Expand-Archive c:/temp/TestApplication.zip -DestinationPath c:/inetpub/wwwroot -force'
    guard_interpreter :powershell_script
    action :nothing
end
```

What you should bring your attention to is namely the way that the powershell_script 'Extract Archive'  has an action of:nothing "which means unless otherwise called; when Chef's processing hits this action; it will do nothing. And then look at the remote_file "Code Source" action. What's worth of note is the notifies attribute. This effectively means "do this when we become aware of a state change rectified by this particular action". So; in essence; we're only going to extract the archive in the event it's changed or was not present, and after it's copied over. All in all; full code to bring up an ASP.NET/IIS webserver, and copy the requisite code over, and update it if/when changed is below.

```ruby
dsc_script 'Web-Server' do
    code <<-EOH
    WindowsFeature InstallWebServer
    {
        Name = "Web-Server"
        Ensure = "Present"
    }

    EOH
end

dsc_script 'ASP.NET' do
    code <<-EOH
    WindowsFeature InstallDotNet45
    {
        Name = "Web-Asp-Net45"
        Ensure = "Present"
    }

    EOH
end

directory 'c:\temp' do
    action :create
end

remote_file "Code Source" do
    source "file:////synology.home.lan//storage//deploy//TestApplication.zip"
    path "C:/temp/TestApplication.zip"
    notifies :run, 'powershell_script[Extract Archive]', :immediately
end

service 'w3svc' do
  action [:enable, :start]
end

powershell_script 'Extract Archive' do
    code 'Expand-Archive c:/temp/TestApplication.zip -DestinationPath c:/inetpub/wwwroot -force'
    guard_interpreter :powershell_script
    action :nothing
end
```

Super simple; definitely not something prepared to scale - but it _does_ give you an idea of the integration between DSC and Chef. 

_I'd like to note that in local-mode Chef-client, the chef-client consistently reports that DSC cannot parse the LCM output. This could be because I'm running on Server 2016 / PowerShell v5; or it could be a quirk in local mode. Regardless, at this time Chef will report a change for every single DSC action, regardless of a change happening._

```
Running handlers:
Running handlers complete
Chef Client finished, 2/9 resources updated in 25 seconds
```

**Next Up:** Ensuring WebDeploy is installed on a host; and using _that_ for application deployment rather than just extracting an archive.