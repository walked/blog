---
title: "Jekyll, Jenkins, and CI"
date: 2017-01-22T12:32:30-05:00
draft: false
tags: ["jekyll", "jenkins" ]
---


Hey guys; as I mentioned - I wanted to come back to run through the workflow I have setup for this blog, CI, and source control!

<!--more-->

So, as I referenced before: I've moved this blog from WordPress / Shared Hosting, to AWS/S3/Cloudfront.

As a part of that, I also revamped my blog publishing workflow and it's been great!

### The components:

1. The blog is build flat-file using Jekyll; Jekyll is more or less a sorta-compiler of sorts that parses style sheets, templates, markdown posts, and asset directories (think images) and compiles it into a nicely formatted flat-file blog, so no server side processing is needed (that all happens at build) and we can use S3 for backend storage, minimize the surface-area of our application, and take advantage of CloudFront. Awesome.

2. The code (markdown files, templates, etc) are stored in git (namely, CodeCommit for me)

3. The build is handled via Jenkins; which has a staging build and a production build. Each performs some transforms and pushes to an S3 bucket accordingly.

### Jekyll:

First off, I'm not going to run through Jekyll in depth. It's pretty quick to pick up. Whats worth noting is on the Jekyll host there are two main commands to know:

```
jekyll build
```

and

```
jekyll serve 
```

Build will simply compile the site into a publishable directory, whereas serve will actually serve the code on a development webserver for testing.

### Jenkins:
This is where it's a bit cooler.

Let's look at my staging job (which is pretty much exactly the same as my production job)

**Quick Summary of the Jenkins Job**

```
General:
├─ Project Name: publish-blog-staging
├─ ☑ Restrict where this project can be run
│     └─ Label Expression: jekyll
│
Source Code Management"
├─ ☑ Git
│     ├─ Repository URL: ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/etcetcetc
│     └─ Branches to build: */staging
│
Build Tiggers:
├─ ☑ Poll SCM
│     └─ Schedule: H/30 * * * *
│
Build Environment:
├─ ☑ Delete workspace before build starts
│
Build:
├─ Execute Shell:  
│     ├─ python3 $WORKSPACE/transform.py --domain="staging.i-py.com" --protocol="http"
│     └─ ~/build-jenkins.bat
│
Post Build Actions:
└─ Publish artifacts to S3 buckets:  
      ├─ Source: _site/**
      ├─ Destination bucket: bucket.domain.com (intentionally generic)
      └─ ☑ No upload on build failure
```

Ok cool. Short and sweet summary:

1. We poll git every 30min; for this project we check the staging branch
2. If a change is detected, it triggers a build.
3. It will clean up the environment workspace prior to build (as we dont have any reason to keep around previous builds)
4. The build executes two shell statements:
   1. ```python3 $WORKSPACE/transform.py``` is a custom transform script I wrote. It will take a domain and protocol argument and edit the Jekyll _Config.yml file prior to build (as my staging site is http://staging.i-py.com I want Jekyll to reflect that on build)
   2. ```~/build-jenkins.bat``` is a super simple script that effectively just calls the ```jekyll build``` command 
5. The site is then uploaded to my Staging S3 bucket (which is where I have my staging site pointed to)

Boom!

Let's look at what my ```transform.py``` script does:

```python 
import argparse, sys, os

parser = argparse.ArgumentParser()
cwd = os.getcwd()
configFile = "_config.yml"

add_arg = parser.add_argument
add_arg("-d",   "--domain",  default="i-py.com",   help="deployment domain")
add_arg("-p",     "--protocol",     default="http",   help="site protocol")
args = parser.parse_args()

if len(sys.argv) > 1:
   print(args.domain)
   print(args.protocol)
   s = open(configFile, 'r+').read()
   s = s.replace("__PROTO__", args.protocol)
   s = s.replace("__DOMAIN__", args.domain)
   file = open(configFile, 'w')
   file.write(s)
   file.close()
```

Really, really simple. It takes a domain and protocol arugment and will asjust my _Config.yml file accordingly. It should be noted I made a change to my config file to look like:

```yml
title: "i-py"
title2: ".com" 
description: "The personal technical blog of Francis Setash. "
url: __PROTO__://__DOMAIN__ # site url
```
_This means I can reuse the same transform script for multiple subdomains (or primary domains)_

### Finally, We Upload to S3

This is using the [S3 Plugin](https://wiki.jenkins-ci.org/display/JENKINS/S3+Plugin_) and really isnt much to discuss. It dumps the contents of ./_site into my Staging S3 bucket; which is viewable at http://staging.i-py.com (dont blame me if you visit and it is broken at some point!)

### Some final notes:
This workflow is duplicated into a second Jenkins job named "publish blog" and really the only differentiation is the branch it looks at, and the parameters it passes the transform script. 

