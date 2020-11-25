---
title: "Chef - dsc_script vs dsc_resource"
date: 2017-01-27T12:32:30-05:00
draft: false
tags: ["powershell", "windows" ]
---

As I continue to experiment with my chef environment, I've run into an interesting limitation with the dsc_script resource.
<!--more-->
As it turns out, with WMF5 RTM, apparently the ```Start-DSCConfiguration``` parsing that chef performs has [broken](https://github.com/chef/chef/issues/5588).

Previously, I had been doing:

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

Unfortunately, with WMF5 - the DSC idempotence is broken. More accurately, _reporting_ on the idempotence is broken.

_Every single chef-client run will report a resource updated for the DSC script_

That's right, every time. Sigh. Even if it's in the desired state prior to the client run.

Fortunately, there is a workaround, the [dsc_resource](https://docs.chef.io/resource_dsc_resource.html) resource. It works with WMF5.0 and properly reports.

```ruby
dsc_resource 'install-sub-features' do
  resource :windowsfeature
  property :ensure, 'Present'
  property :name, 'GPMC'
  property :IncludeAllSubFeature, true
end
```
The downside, here, is that it's a bit less readable. I actually have a strong preference for native DSC scripts; but this will work for now - its not _too_ farr off DSC verbiage.

Anyways; just a quick tidbit. If you're running WMF5.0 and Chef - you should favor ```dsc_resource``` over ```dsc_script```.

