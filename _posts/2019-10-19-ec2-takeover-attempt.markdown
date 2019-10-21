---
layout: post
title:  "Attempting EC2 Subdomain Takeover"
date:   2019-10-19 10:39:12 -0700
categories: ec2 deadend
---
So there I am, doing recon on a Hackerone private program. Five reports in the last 90 days seems like either an ignored target or a hardened target (jury is still out on that one).

## What I Found
The first step of my recon is to find all the subdomains that are in scope, which I think is pretty standard at this point. I then grabbed the DNS records for all of these subdomains, hoping for some easy subdomain takeovers. A quick `grep` revealed some CNAME records, which is a good signal that subdomain takeover is a possibility. Some of the records were pointing to a very familiar-shaped URL: `ec2-192-168-1-1.us-west-1.compute.amazonaws.com` (changed, obviousy). I recognized this as the "Public DNS" of an EC2 instance. All in all there were about 10 of these entries.

This seemed a little weird to me, my understanding is you don't usually point domains to EC2 instances directly, you use ELB or an ElasticIP or many other mechanisms. Part of the reason for this is when you stop an EC2 instance, it's IP address changes, so a CNAME pointing to that EC2 instance's public DNS entry would no longer work.

Now I was curious, what happens when you provision an EC2 instance (getting a Public DNS address) and then stop that instance? A quick test reveals that HTTP requests timeout when you request the Public DNS of a stopped EC2 instance.

Next, I checked the content on the domains. Half served the correct content, half of them timed out. Oh this feels juicy. Half of these might be pointing to EC2 instances that anyone could spin up!

### What I Tried

At this point I'm hopeful that I can get control over these EC2 instances and get multiple subdomain takeovers. The only question was how to get control of these EC2 instances? After some Googling it looked like there was no way to request a specific IP address for an EC2 instance. Well, I thought, this makes things harder, but not impossible. So I wrote a terrible little script.

{% highlight bash %}
while true; do
  aws ec2 stop-instances --instance-ids $INSTANCE_ID
  sleep 30
  aws ec2 start-instances --instance-ids $INSTANCE_ID
  sleep 30
  aws ec2 describe-instances --instance-ids $INSTANCE_ID | grep ec2-192-168-1-1.us-west-1.compute.amazonaws.com

  if [ $? -eq 0 ]; then
    echo "DNS Target Hit"
    exit 0
  fi
done
{% endhighlight %}

Basically, every stop/start cycle gives me a new IP and Public DNS address, it's then checked against the DNS address I'm trying to gain control of. If it's found, exit the script. To be honest, kicking that script off made me a little nervous that AWS would rate limit and/or kick me, but nothing like that happened.

So I let my script run.
... and run.
... and run.

After a long time of getting no hits (and yes I know the odds of hitting the right IP addresses are really low) I decided to revist my assumptions.

### What I Missed

I went back to those original DNS records, and I noticed a detail that I skipped at first. The DNS records were all for distinctly internal subdomains. Names like `internal` and `dev-cluster`. To me, this greatly increases the chances that the the EC2 instances ARE under the control of the correct company, but they've placed IP/port restrictions on them that makes them inaccessible from the outernet.

### Conclusion

All in all this wasn't a total failure because I did learn a fair amount and there is still a chance that these EC2 instances could be claimed by anyone.

I hope you enjoyed this first entry in what may become my bug hunting fail blog. Feel free to reach out on Twitter (@MurmurPanda) if I missed anything or you have feedback.

