---
layout: post
title: "How we debugged rails memory leak"
date: 2013-01-19 13:29
comments: true
categories: 
---

We run our app on heroku and kept on getting R14s. Frustrating..

* Figured out what pages create R14
    * Observe pattern of usage
    * Load test app with multiple usage paths. Figure out which falls soon.
<p>This is more of asking more questions to users and making guesses. We had a page that was most frequently used. We did a 
load test on that page and within in minutes it ended up with R14.</p>
* Replicated the leak in local machine
    * Run the app in production environment
    * Use <code>vmmap</code> to find out memory usage pattern. (You can use smap or pmap if you are using linux.)
              #!/bin/bash
              while :
                do
                  echo `vmmap $PROCESS_ID | grep ^DefaultMallocZone_0x`
                  sleep 1
                done
    * The o/p looks like this.
