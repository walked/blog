---
title: "Chef - Bootstrapping Windows in Service Mode"
date: 2017-02-01T12:32:30-05:00
draft: false
tags: ["windows" ]
---

So; ```chef windows bootstrap``` doesnt have an install as service option. Dang it; that would be too convenient. Even the official docs dont say anything about it..

<!--more-->

[Here](https://docs.chef.io/plugin_knife_windows.html) is the official documentation.


## Quick snippet:

```
-G GATEWAY, --ssh-gateway GATEWAY
    The SSH tunnel or gateway that is used to run a bootstrap action on a machine that is not accessible from the workstation.
-i IDENTITY_FILE, --identity-file IDENTITY_FILE
    The SSH identity file used for authentication. Key-based authentication is recommended.
-j JSON_ATTRIBS, --json-attributes JSON_ATTRIBS
    A JSON string that is added to the first run of a chef-client.
-N NAME, --node-name NAME
    The name of the node.
--[no-]host-key-verify
    Use --no-host-key-verify to disable host key verification. Default setting: --host-key-verify.
```

Right. Ok.

That said; [here's a _very interesting_](https://github.com/chef/knife-windows/pull/215) github pull request:


```ruby
             :description => "Bootstrap a distro using a template",
              :default => "windows-chef-client-msi"
  
 +          option :install_as_service,
 +            :long => "--install-as-service",
 +            :description => "Install chef-client as service in windows machine",
 +            :default => false
 +
            option :template_file,
              :long => "--template-file TEMPLATE",
              :description => "Full path to location of template to use",
```

and

```ruby
         private
  
          def install_command(executor_quote)
 -          "msiexec /qn /log #{executor_quote}%CHEF_CLIENT_MSI_LOG_PATH%#{executor_quote} /i #{executor_quote}%LOCAL_DESTINATION_MSI_PATH%#{executor_quote}"
 +          if @config[:install_as_service] || @knife_config[:install_as_service]
 +            "msiexec /qn /log #{executor_quote}%CHEF_CLIENT_MSI_LOG_PATH%#{executor_quote} /i #{executor_quote}%LOCAL_DESTINATION_MSI_PATH%#{executor_quote} ADDLOCAL=#{executor_quote}ChefClientFeature,ChefServiceFeature#{executor_quote}"
 +          else              
 +            "msiexec /qn /log #{executor_quote}%CHEF_CLIENT_MSI_LOG_PATH%#{executor_quote} /i #{executor_quote}%LOCAL_DESTINATION_MSI_PATH%#{executor_quote}"
 +          end
          end
  
          # Returns a string for copying the trusted certificates on the workstation to the system being bootstrapped

```

_hmmmmmmm_

Let's see if it works.

```> knife bootstrap windows winrm sts601-dsc  -r 'baselineRecipe' -x 'DOAMIN\USER' -P NO_PASSWORD_HERE --install-as-service```

IT DOES!

Huzzah!

_Edit:_ Submitted a [pull request](https://github.com/chef/chef-web-docs/pull/438) to update the Chef documentation