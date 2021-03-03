---
layout: post
title:  "DOM XSS is Dead*, Long Live DOM XSS"
date:   2021-02-23 12:00:00 -0700
categories: xss dom burp
---

*With the notable exception of `postMessage`

## Intro

For the past couple of months I've been spending some of my spare time studying DOM XSS. I wanted to do a deep dive on a single vulnerability sub-type so that I can master it, add it to my toolbox of things to look for during bug bounty hunting, and move on. Things didn't really go according to plan.

For the sake of brevity I'm not going to give a thorough explanation of what DOM XSS is and how it contrasts with other XSS types. The short version is this: DOM XSS is when the Javascript of a page uses attacker set-able client side data sources in an unsafe way.

## Finding DOM XSS

DOM XSS requires client side javascript to have logic that takes a vulnerable source (attacker controllable value) and passes the value to a vulnerable sink (function or property with security implications). In general, there are 3 ways of finding DOM XSS
1. Manual code review ("bottom up" search)
  - made harder by code minimization, obfuscation, and packing
  - time consuming
  - error prone
2. Populate sources and look for test strings appearing in the DOM ("top down" search)
  - easy to miss vulnerable code paths
  - time consuming
3. Code scanners (static or dynamic)
  - sometimes costly licenses
  - results not guaranteed

Of these 3 options, code scanners are the most viable option. There are many scanners out there, but I didn't feel like paying the subscription fees to try them all out. Some were ludicrously expensive for my needs (up to $999/year). For that reason the only scanner I tested was Burp Suite Pro (the license I already pay for).

Update: There are tools that assist in manual searches for DOM XSS like [@filedescriptor's](https://twitter.com/filedescriptor) [untrusted-types](https://github.com/filedescriptor/untrusted-types).

## Sources

A Source is any property accessible via Javascript that *could* be set-able by an attacker. Sources aren't as simple as we want them to be and I couldn't find any consistent language for the different types. For this post I've separated them into "True Sources", "Sunk Sources", and "Encoded Sources".

### True Sources

True Sources are the sources that an attacker is well and truly able to set without needing to chain with anything else (like user interaction or controlling the value through DOM manipulation) and whose value is not automatically sanitized or encoded. The "DOM XSS is dead" title comes from the most common DOM XSS sources now being URL Encoded by default in all major browsers. The list of True Sources is depressingly short:

- `window.name`
- `message` event listener
- `document.URLUnencoded` (IE only, not personally verified)

### Sunk Sources

Sunk Sources are sources which cannot be set by the attacker by default, but are sometimes set-able through site specific client or server behaviour.

For example, `document.cookie` is a Sunk Source because an attacker cannot normally set this value for a victim, making it not really a source. However, the server may do something like return a cookie that includes the referring URL (an Encoded Source), or the client side Javascript could include `document.cookie = location.hash` (also an Encoded Source). Both of these behaviours would make `document.cookie` a viable source for a DOM XSS attack. This introduces a lot of complexity to searching for DOM XSS since it's hard to definitively determine whether a given Sunk Source is set-able elsewhere. When working on an exploit chain, always use your browser console and debugger to make your life easier.

List of Sunk Sources:

- `document.cookie`
- `history.pushState`
- `history.replaceState`
- `localStorage`
- `sessionStorage`
- Browser integrated databases (mozIndexedDB, webkitIndexedDB, msIndexedDB)
- `XMLHttpRequest` responses (if the target URL is controlled by the attacker)

### Encoded Sources

Encoded Sources are the worst because they used to be True Sources but browsers had to go ahead and ruin the fun for everyone. All up to date modern browsers URL encode these sources which make them mostly useless (I have yet to find a DOM XSS where an Encoded Source is decoded before being passed to a sink). Many resources on DOM XSS have not been updated to reflect these browser changes.

List of Encoded Sources:

- `document.URL`
- `document.documentURI`
- `document.baseURI`
- `location.*`
- `document.referrer`

## Sinks

Sinks are functions or properties that could have security impacts if controlled by an attacker and are defined by the attacks possible when an arbitrary string is passed to them. This post does not have an exhaustive list of sinks by attack type, for that the best source is [PortSwigger Academy](https://portswigger.net/web-security/dom-based) although I disagree with some of their classifications/scan results.

Below are lists of sinks that are either misclassified by Burp Suite, have specific requirements to be exploitable, or I felt like giving some extra context on:

### Javascript Injection

- `Function () constructor` - Burp Suite will only find this if the source is passed to the first argument, which misses vulnerable code
- `document.execCommand` - requires `document.designMode === "on"`
- `setImmediate` - Only available in IE, not verified
- `*.createContextualFragment` - rare, this is a function on `range` objects created by `document.createRange`,
- `location.assign()` and `location.replace()` - escalate from Open Redirect to XSS by using the `javascript:` protocol
- `jQuery.globalEval()` / `$.globalEval()` - PortSwigger Academy has these listed under "Ajax request-header manipulation" and I don't know why
- `iframe.srcdoc` - HTML that overrides an iFrame's src attribute, possible to get Javascript Injection
- `script.src` - probably rare for the whole src string to be controllable, but if you find it you could load any JS file you want

### Ajax Request Header Manipulation

- `XMLHttpRequest.open()` - If the URL parameter is a sink then any sensitive headers attached to the request could be read by a malicious server
- `XMLHttpRequest.send()` - I don't know how this is a sink, it only accepts one parameter (body) and I couldn't figure out how this is exploitable

## Scanning with Burp Suite Pro

My experience with Burp Suite Pro was largely positive. There are some sinks or sources that give misleading information, but almost all direct links from Source -> Sink were found. Some of the Sunk Sources are treated as sinks by the scanner rather than sources in their own right. For example, it will find `hash` (Encoded Source) being passed to `localStorage` (Sunk Source), but it won't catch `localStorage.getItem(key)` (Sunk Source) being passed to `document.write()` (Sink). Another drawback is how Encoded Sources are treated as True Sources which causes a lot of false positives.

## Conclusion

"DOM XSS is dead" is hyperbole (why did I make a clickbait title? This blog doesn't have ads), a much more honest title would be "Simple DOM XSS is dead". I believe there are still a lot of DOM XSS issues out there, but finding them will require a combination of Sunk Sources, the `postMessage` API, and tons of creativity. There's no limit to how many links in the chain there could be from source to sink, and if you can thread that needle you'll be rewarded. Get gud at reading Javascript because scanners will only give you clues about individual steps; in a full exploit chain, you'll have to do the heavy lifting yourself.

There's still a lot out there to find, best of luck in the hunt, friends!
