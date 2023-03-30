---
title: "One In Two New Npm Packages Is SEO Spam Right Now"
datePublished: Thu Mar 30 2023 08:56:30 GMT+0000 (Coordinated Universal Time)
cuid: clfuvt2jy000a09jreaotd5a9
slug: one-in-two-new-npm-packages-is-seo-spam-right-now
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680171463954/6154fb15-a996-4c1a-b61a-af5dfe1db606.jpeg
tags: security, seo, npm, spam

---

**More than half of all new packages that are currently (29 Mar 2023) being submitted to npm are SEO spam.** That is - empty packages, with just a single README file that contains links to various malicious websites.

%[https://datawrapper.dwcdn.net/ZkWKj/2/] 

Out of the ~320k new npm packages or versions that [Sandworm](https://sandworm.dev) has scanned over the past week, at least ~185k were labeled as SEO spam. **Just in the last hour as of writing this article, 1583 new e-book spam packages have been published.**

All the identified spam packages are currently live on [npmjs.com](https://www.npmjs.com/).

Here's a breakdown of the main attacker profiles for the week's worth of data we're sampling in this article (22-29 Mar 2023):

%[https://datawrapper.dwcdn.net/w4jWt/4/] 

## The Russian Telegram Channel

Most of the spam packages detected by Sandworm come from a single Telegram channel that seems to be targeting Russian-speaking people. Package names are set to match searches on various sensitive topics, like the war in Ukraine or investment decisions made by Gazprom. The package description, however, reads:

> *Forget about financial problems forever: a new method of earning will allow you to earn millions without leaving your home!*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680120051492/affa6d03-ddef-4241-99a2-1ad24c4d0cc3.png align="center")

The links point to the malicious Telegram channel, with over 7k members. All 93k+ packages analyzed have this identical Telegram URL in their description.

## The Classics: Free Books/Videos

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680120301454/e0725267-3ebb-445e-8b28-bfdfd015c3b5.png align="center")

The second biggest SEO spam actors on npm are the more conventional ads for free books and videos online. These lead to websites that require the user to perform a series of tasks to obtain a (non-existent) download link, basically watching and interacting with ads. The domains used in URLs by these spam packages change a lot, so they're harder to detect.

## Fortnite V-Bucks

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680120721695/809e7536-c931-4ac5-a28f-8a100e90881b.png align="center")

A new trend is fake packages promising free Fortnite V-Bucks. These usually take users to elaborate pages where they're tricked into thinking they need to verify they're human by performing a series of tasks - again, ads and surveys.

## Sandworm's Response

We're in the process of reporting all of the identified spam packages to npm. We suspect this is only the tip of the iceberg since we've been able to identify many packages that have been live in the npm repo for years (like `uyo-xint`). We'll have a deep look into spam across the entire registry in a future blog post.

[Sandworm](https://sandworm.dev) continuously monitors the npm registry. We post updates and investigate new software supply-chain threats as soon as we identify them. Like, follow, and subscribe to our newsletter for more!

And keep your JavaScript project safe with Sandworm 👇

%[https://github.com/sandworm-hq/sandworm-audit] 

> [*Sandworm Audit*](https://github.com/sandworm-hq/sandworm-audit) *is the open-source* `npm audit` *that doesn’t suck: it checks for multiple types of issues, like vulnerabilities or license compliance, it outputs SVG charts and CSVs, it can mark issues as resolved, and you can also run it in your CI to enforce security rules.* [*Check the docs*](https://docs.sandworm.dev/) *and* `npx @sandworm/audit@latest` *in your JavaScript app’s root to try it out 🪱.*
> 
> ![](https://cdn-images-1.medium.com/max/2120/1*JvTQJzapGNSSCKWQi89hCA.gif align="center")