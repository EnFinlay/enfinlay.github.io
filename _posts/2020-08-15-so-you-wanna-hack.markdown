---
layout: post
title:  "So You Wanna Hack"
date:   2020-08-15 15:50:23 -0700
categories: bugbounty hack beginner noob starter
---

Over in the Hacker101 Discord, a half dozen people come in every day asking "how do I hack?". The mods have done a great job helping answer this question with pinned messages, a `!beginner` macro, and a welcoming atmosphere. This post is for those who want to get into ethical hacking / bug bounty, but have no idea where to start and need more technical background before they can start grokking the introductory hacking resources.

## Step 0: Mindset

- "Hacking" is a collection of dozens of fields each with many specializations. No one is proficient in every area of "hacking" and therefore no one can "teach you hacking". The rest of this post will refer to the bug bounty subset of hacking / infosec.
- The most important skills are patience, research, and self-directed learning. There is no resource to answer every question you have, **You Must Figure Things Out For Yourself**
- There are a ton of resources out there already and a quick google / follow party on Twitter will lead you to articles, practice environments, and tools. Most of it will require a bit of leg work on your part to understand fully. Do that work, it's worth it.

## Step 1: The Internet

You need to understand how the internet works at a high level. This does not mean becoming an expert or reading every single RFC (although it won't hurt). A rudimentary understanding of the following is required:
- [OSI Model](https://en.wikipedia.org/wiki/OSI_model)
- [DNS](https://en.wikipedia.org/wiki/Domain_Name_System)
- [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)

Some of this won't make sense on first read, but that's okay. Read through things multiple times, research the questions you have, and take notes. Don't stop at just one read through and don't spend the next 6 months exploring every nook and cranny. Be able to explain it to someone with only a couple hand wavy "and then something happens" moments.

## Step 2: Websites

After you have a good idea of how the internet is meant to work, you need to become familiar with your most common target: websites. I don't have any great resources for this, but MDN is one of the best references on the internet so [this guide will probably do you good](https://developer.mozilla.org/en-US/docs/Learn).

Focus on HTML and Javascript and how they work together to create the dynamic experience that we're used to. Don't just read - experiment. You can run a local webserver from your computer with python `SimpleHTTPServer`.

## Step 3: Security Vulnerabilities

Congratulations! You now have enough background knowledge to begin your security learning. The best first resource I can recommend is [hacker101.com](https://www.hacker101.com/) and the accompanying videos. The videos are modern enough to be relevant and created by great hackers. Notice that I'm not giving a big list of resources because that choice doesn't help right now. Watch the videos. Google and understand the things that confuse you. Come to Hacker101 Discord and ask questions. (Aside: asking "I don't understand what Cody said about the HTTPOnly flag at this timestamp of this video" is 10x better than "This is confusing, someone explain it to me").

## Step 4: Everything else

In no particular order here are some things that will help you

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Hackerone Hacktivity](https://hackerone.com/hacktivity)
- [PortSwigger Academy](https://portswigger.net/web-security)
- [Nahamsec's Streams](https://www.twitch.tv/nahamsec)
- [InsiderPHD's Youtube Channel](https://www.youtube.com/channel/UCPiN9NPjIer8Do9gUFxKv7A)
- [Stök's Vibes](https://www.youtube.com/channel/UCQN2DsjnYH60SFBIA6IkNwg)

Each one of these will lead you down a nearly infinite rabbit hole so buckle up.

## Conclusion

Learn the basics, practice them on VDPs (Vulnerability Disclosure Program - no bounties), and stay curious.

Take it slow. Slow is steady, steady is smooth, smooth is fast.

Good luck and happy hacking!
