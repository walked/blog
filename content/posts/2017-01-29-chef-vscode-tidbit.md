---
title: "ChefDK - Visual Studio Code Support"
date: 2017-01-29T12:32:30-05:00
draft: false
tags: ["powershell", "windows" ]
---
Super quick post, but one I wanted to throw down - here's how to get Visual Studio Code working for the ChefDK in Windows; and I'll also share my Visual Studio Code preferences file!

<!--more-->

So, this should hold true for using VS Code as a native/default editor for Git and any other application with a like workflow.

## knife.rb
```ruby
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "myWorkstationNode"
client_key               "#{current_dir}/username.pem"
validation_client_name   "org-validator"
validation_key           "#{current_dir}/org-validator.pem"
chef_server_url          "https:/mychefserver/organizations/ipy"
syntax_check_cache_path  "#{ENV['HOME']}/.chef/syntaxcache"
cookbook_path            ["#{current_dir}/../cookbooks"]
knife[:editor] = '"C:\\Program Files (x86)\\Microsoft VS Code\\Code.exe" --wait'
```

The last line is what needs to be noted;
- Backslashes must be escaped ```\\``` 
- Because there is a space in the filepath, it must be surrounded by single quotes enclosing double quotes. ```' " " '```
- The ```--wait``` paramater is critical otherwise ChefDK will get an immediately executed command for ```Code.exe``` and see it as a file which was never saved.

And then, becuase I've been working with VS Code a while; here is my Visual Studio Code User Preferences that I think others may like:

```json
{
    "editor.cursorStyle": "block",
    "terminal.integrated.shell.windows": "c:\\windows\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe",
    "terminal.integrated.scrollback": 3000,
    "editor.renderWhitespace": "boundary",
    "editor.renderIndentGuides": true,
}
```

Let's see here:
- ```"editor.cursorStyle": "block"``` Because I've found it _much_ easier to work with a block style cursor for editing code.
- ```"terminal.integrated.shell.windows": "c:\\windows\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe"``` - need I say more? I'm actually surprised this isnt the default.
- ```"terminal.integrated.scrollback": 3000,``` - because sometimes it's nice to have and there's almost no downside.
- ```"editor.renderWhitespace": "boundary"``` Honestly because it's a must have setting for the next one to have any effect..
- ```"editor.renderIndentGuides": true,``` An alternative is the Indent Rulers option; but I find this less obtrusive and just as useful for lining up code.

Really not a ton that needs to be tweaked, but I find these minimal options make the editor MUCH more pleasant for me to use.

