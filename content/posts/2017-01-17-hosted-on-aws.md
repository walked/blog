---
title: "This blog has been migrated to AWS"
date: 2017-01-17T12:32:30-05:00
draft: false
tags: ["aws", "jekyll"]
---

Over the holiday weekend I've made some major changes to this blog! _Retroactive apology to anyone who had bookmarked anything on this site!_

<!--more-->

Prior to this weekend; I've been running a pretty hacked version of WordPRess on a budget host. Frankly, it was mostly fine but I'd noticed some downtime here and there, and once in a while they'd just be _slow._ Overall for the price I was paying, they were great.

However, I finally made the jump to move off WordPress and to Jekyll. 

Overall I'm really pleased with the move. Flat-file; no need for a DB for a site that ostensibly does not need it. Markdown for blog posts. Integrates with CI (I'll put together another post later, but I'm running a full blown CI setup with Jenkins, Git, and AWS - really happy)

### So how's the site setup?

- **Files:** AWS S3 Bucket Setup for Static Page Hosting
- **CDN:** AWS CloudFront is acting as the CDN for this site; lightning fast!
- **DNS:** Now using Route53 instead of NameCheap's DNS services
- **CI:** Jenkins, polling a Git repo hosted in AWS CodeCommit; ```master``` branch is the production site - built automatically on any commit to the master branch; ```staging``` branch is published to my staging site on every commit to the staging branch

No code to share today; more or less just a post to say "This blog has changed a bit!" and I'll do a writeup of configuring Jenkins for CI in tandem with Jekyll in the near term.