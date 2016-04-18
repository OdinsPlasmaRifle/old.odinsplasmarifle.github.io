---
layout: post
title: X Robots Tag In Apache Headers
comments: true
---

*X-Robots-Tag HTTP header response for a given URL*

I recently had one of my clients request that I set "noindex nofollow" meta tags on a specific set of pages. Unfortunately, these particular URLs were AJAX templates and therefore didn't have a 'head' element I could easily insert meta tags into. While investigating, I discovered a way to set headers that achieve the same things as robots meta tags. This can be done with the X-Robots-Tag headers:

```
HTTP/1.1 200 OK
Date: Tue, 25 May 2010 21:42:43 GMT
(…)
X-Robots-Tag: noindex
(…)
```

<!--break-->

## The Apache problem

I quickly realized this would be a much more elegant way to handle meta tags on large portions of a website. For instance, on a multitude of file types or even on a URI like `/ajax/`.

When using an Apache Virtual Host or htaccess file the first use-case (header based on file type) is simply achieved like this:

```
<ifModule mod_headers.c>
   <FilesMatch "\.(pdf|flv|jpg|jpeg|png|gif|swf|ttf|eot|svg)$">
        Header set X-Robots-Tag "noindex, nofollow"
    </FilesMatch>
</ifModule>
```

However, it gets a little more complex when trying to do the same thing on a URI "directory" like `/ajax/`. I could not match on the `<Directory>` Apache element because that makes use of the system's file structure rather than the website URI.

I spent a long time digging around and then realized an Apache [envif](https://httpd.apache.org/docs/current/mod/mod_setenvif.html) could probably be used to achieve this. As a result I came up with the following solution:

```
<ifModule mod_headers.c>
	# Set Ajax Headers
	SetEnvIf Request_URI "/ajax" AJAX_HEADER
	Header set X-Robots-Tag "noindex, nofollow" env=AJAX_HEADER
</ifModule>
```

It is quite straight-forward actually. Firstly, we set a variable `AJAX_HEADER` if a Request_URI matches `/ajax/`. In the next line a header is set IF the above variable has a value. This way we essentially have a conditional that checks for a certain URI string and then sets a header if that is true.

Hope this spares someone the effort of browsing through stack overflow and dozens of infamous Apache documentation pages.

### Further reading:

Take a look [here](https://moz.com/learn/seo/robotstxt) if you are curious about the difference between meta robots and the robots.txt disallow keyword.

[Google Robots Docs](https://developers.google.com/webmasters/control-crawl-index/docs/robots_meta_tag)
