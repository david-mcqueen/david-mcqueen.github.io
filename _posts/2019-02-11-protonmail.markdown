---
layout: post
title:  "ProtonMail - Encrypted Mail"
date:   2019-02-16 11:51:00 +0000
categories: blog posts
---

## Privacy Matters

I enjoy personal privacy. Who wouldn't? We all have our secrets, things we don't want the world to know, things which are better left un-told and not shared on _social media_. Things such as your financial state, your medical history, or a teenage pregnancy, for example (Not me!). Unfortunatly we live in a world where privacy is far too often traded away to corporations for 'free' services. Services where **the consumers are the products** and **the advertisers are the customers**, such as Facebook, Snapchat, and Google. [Data has surpassed Oil as the worlds most valuable resource](https://www.economist.com/leaders/2017/05/06/the-worlds-most-valuable-resource-is-no-longer-oil-but-data) and with it, corporations are able to do things which at one point we would only have been able to dream about, such as [figuring out that teenage pregnancy](https://www.forbes.com/sites/kashmirhill/2012/02/16/how-target-figured-out-a-teen-girl-was-pregnant-before-her-father-did/) before the family knows - possible because of how much data the corporation has on _millions_ of people, including you. Individually you buying something slightly out of the normal is an anomaly, but when compared to millions of other data points (people), conclusions can be drawn. Pregnant people buy _x y z_, thus you are pregnant because you also bought _x y z_, for example.

Amazing that it is possible to do, but also concerning when revelations emerge that such techniques have been used to manipulate the Brexit vote ([Cambridge Analytica](https://www.theguardian.com/news/series/cambridge-analytica-files)) and the Trump / Clinton Presidential Election, just to name a couple.

## Encrypted Mail
When it comes to email we're not uploading a new "status update", declaring our undying love for avocado on toast, for the corporations to harvest, but we are letting them read all of the emails sat in our mailbox. When you're messages are unencrypted and sat on a server (which is likely free to you) **you are the product**. The hosting provider is reading your messages, and knows more about you then you would likely want. It's akin to your postman reading every letter he puts through your door, and then telling his friends.

This is what has drawn me to ProtonMail. Your messages are stored with [Zero-Access](https://protonmail.com/blog/zero-access-encryption/) meaning that the provider cannot read what is sat in your inbox - it's private and your postman has no idea what he is delivering. 

Until recently I've been using Gmail, and have been paying for it (because I wanted a custom domain), letting them do what they want with my data. However they have announced that the price is going to be increasing - just the little extra push I need to make the move!


## Adding a Custom Domain
One thing which kept me from switching from Gmail to ProtonMail, after the initial setup, was the concern of downtime over the switch period - I didn't want to miss any emails, especially those which could be important, and there never seemed to be a good time to be able to do the switch. 
But fear not! It is easier than making a banana split...

Ok maybe it's a little bit more involved than a banana split, but its still incredily easy! It is all pretty well [documented on the ProtonMail site](https://protonmail.com/support/knowledge-base/transitioning-from-gmail-to-protonmail/). I'm not going to go into detail on all of the steps, as there is no point in repeating what ProtonMail has already covered, but I do want to talk a little bit about a couple of areas I found interesting.


### Don't modify your Gmail settings
First thing first, leave these alone! For now at least. We still want Gmail to receive your messages until ProtonMail is fully setup and ready to take over, this will ensure that no messages bounce back to the sender. We're basically going to setup ProtonMail to be able to receive messages sent your your email address, and then update the signpost (_MX_ DNS records) telling messages to routes to ProtonMail instead of Gmail.

### Adding a custom domain & Modifying the DNS records
After telling ProtonMail about your custom domain, it'll take you through a _wizard_ which tells you which DNS settings to configure. I use [CloudFlare](https://www.cloudflare.com/) with all of my domains, so I needed to modify the settings within Cloudflare, as opposed to my Domain Registrar, to setup the DNS records.

In-depth detail for setting up and verifying a custom domain can be found on the [ProtonMail Blog](https://protonmail.com/support/knowledge-base/dns-records/).

Along with the core _MX_  DNS settings, ProtonMail also gives you the option to set a couple of other settings, namely SPF, DKIM, DMARC.

![Protonmail Domain Settings](/assets/images/protonmail/protonmail-domain-settings.png)

In the image above you can see that the boxes for each of these are all green for my domain, showing that they have been configured.


## Anti-Spoofing

SPF, DKIM, and DMARC are all designed to protect your domain name from spoofing. It's fairly trivial to change the displayed name on any outgoing email to anything you want it to be, meaning that if they wanted to, someone would be able to send a message claiming to be you. Not ideal. These 3 settings help to combat that.

### SPF
SPF (Sender Policy Framework) dictates who is allowed to send emails on your behalf. When the recipients email server receives an email claiming to be from you, it checks your DNS records for who is allowed to send emails, based on the SPF settings. If it is a different sender server, then it is a possible cause for concern.

### DKIM
DKIM (Domain Keys Identified Mail) verifies the integrity of an email. When you send a message, ProtonMail will calculate a hash for the message using a private key. Upon delivery of the email, the recipient email server will lookup the public key (set in your DNS settings) and calculate another hash. If the two are different then it is a cause of concern - something in the message content changed during transit.

### DMARC
DMARC (Domain-based Message Authentication, Reporting, and Conformance) is a setting which works in collaboration with SPF & DKIM. It defines what should happen to messages which are received by an email server, and fail either SPF or DKIM verification. On the lowest end of the settings you can tell the server to do nothing, and deliver the messages like normal, and on the other end of the settings you can tell the email server to discard the messages as spam. You'll receive an email in your inbox with an XML report on how an email server has handled a message claiming to be from you, and if it passed SPF and DKIM verification;

![DMARC Report](/assets/images/protonmail/dmarc-report.png)

## DNS Configuration

At the end of the configuration, you should have something similar to the following DNS settings;

![Cloudflare DNS](/assets/images/protonmail/cloudflare-dns.png)

This consists of 2 MX entries - defining the 2 servers which should be tried for routing new mail to - and 4 TXT entries; A *protonmail_domainkey* verification entry to verify you own the domain name, and the anti-spoofing entries for SPF, DKIM, and  DMARC.

## Summary

I would reccommend looking at yet another [ProtonMail blog](https://protonmail.com/support/knowledge-base/anti-spoofing/) which goes into a lot more detail for Anti-Spoofing DNS settings. It's a pretty interesting read and gives a bit more insight into email as a whole.


Following that, I would highly reccommend getting yourself a [ProtonMail](https://protonmail.com/) account, its free if you are happy to use an _@protonmail.com_ address, plus it actually respects your privacy. On top of that, it allows for PGP encryption when sending emails, meaning that the entire message content is encrypted and thus can only be read by the sender, and the recipient. Pretty nifty.

I should probably mention that there are other email providers out there which offer similiar services, it just so happens that ProtonMail is the one that I use and think provide a good service. I'm just a bit of a fanboy, apparantly.

Now I'm going to go read 1984...
