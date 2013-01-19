---
layout: post
title: "How we debugged rails memory leak"
date: 2013-01-19 13:29
comments: true
categories: 
---

We run our app on heroku and kept on getting R14s. Our google and stackoverflow skills did provide
few suggestions and but din't help much. We used few tools (TODO) see the memory usage. They provided who use how much
but we could make further progress with that. So here's what we did to find out.

* Figured out what pages create R14
    * Observe pattern of usage
    * Load test app with multiple usage paths. Figure out which falls soon.
<p>This is more of finding usage patterns and making guesses. We should ascertain our guess with heavily loading the suspected page(s) and see the memory pattern. We had a page that was most frequently used. We hit it with jmeter and within in minutes it ended up with R14.</p>
* Replicated the leak in local machine.
    * Ran the app in production environment and kept hitting the page.
    * Used <code>vmmap</code> to find out memory usage pattern. (You can use _smap_ or _pmap_ as well.)
    * Saw the memory usage keep on increasing and never getting stabilized.
<p> When we start the rails app and start visiting pages, the memory goes up. It increases to certain extent and gets stable after
a while. Don't be misguided by the initial increase in memory as memory leak. It happens for all the pages. But, it gets stable soon. For leaking pages, the memory never gets stabilized.
</p>
* Drilled down and found the code that leaks.
    * Removed the action and view code and rendered the page and saw memory was stable. This is to ensure
    the problem is with the specific page and not the framework as such.
    * Added code step-by-step in chunks and observed memory usage.
    * Figured out which chunk leaks the memory.

<span> Finally, nailed down to <code>model.has_warnings?</code>.</span>
We use validation_scopes which was the cause in our case. It was creating metaclasses for validation
and those classes did not get garbage collected.

We fixed it by replacing _validation_scopes_ with _activemodel-warnings_.