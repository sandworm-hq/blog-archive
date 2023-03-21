---
title: "The Better Npm Audit 🪱"
seoTitle: "The Better Npm Audit - Security & License Compliance CLI"
seoDescription: "With Sandworm Audit, you're now able to take ownership of your security workflow, just like you always wanted: with free, open-source tools!"
datePublished: Sun Mar 19 2023 14:38:35 GMT+0000 (Coordinated Universal Time)
cuid: clffi6lxa00010al13w7beu4c
slug: the-better-npm-audit
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679140680190/2997bfe9-7795-48a7-b1bf-9e0d4a12ebb7.png
tags: javascript, security, command-line, license, compliance

---

It's been almost two years since Dan Abramov wrote his controversial article on how `npm audit` is [broken by design](https://overreacted.io/npm-audit-broken-by-design/):

> **It makes experienced app developers miserable** because they have to either waste time doing obviously unnecessary work, or fight with their security departments trying to explain how `npm audit` is a broken tool unsuitable for real security audits *by design*.

Two years - yet not much has changed when it comes to the number of false positives you get from `npm audit` - false positives that have made JavaScript developers learn to ignore all security warnings in the console when installing new packages. Ironically, making poor audit results part of the default output when installing packages has had a reverse effect, by effectively training developers to ignore security warnings they can't easily understand or address.

## It's hard to enforce security rules

So let's say you're not that guy. You care about security and see the threat in blindly relying on hundreds - often thousands - of packages with 3rd party code. You don't currently have the budget for the better-known services out there, but you still want to build security into the core of your application and your workflows as much as possible.

So you add `npm audit --audit-level=high` to your CI pipeline to enforce "no builds and/or deploys when there are high or critical vulnerabilities". Then you install some dependencies, write some code, and push an update. The build fails because [`d3-color<3.1.0` is vulnerable to ReDoS](https://github.com/advisories/GHSA-36jr-mh4h-2g58). It's high severity so you have a look, and it turns out you're not using any of the options that can allow malicious input to [make some RegExp explode](https://github.com/d3/d3-color/pull/100). Great, there's nothing to address here! But what about fixing the build now? 🤔

## Not all issues are resolved with code

Ok, so maybe you remove the audit step from the CI for now but start using [Dependabot](https://github.com/dependabot). It surfaces the same vulnerabilities that your package manager would, directly from [GitHub's Advisory Database](https://github.com/advisories). It makes PRs with fixes, when available, and you can dismiss alerts that don't apply by selecting one of a few possible reasons:

![Dependabot dismiss options. You can't specify more than this.](https://cdn.hashnode.com/res/hashnode/image/upload/v1679094191780/d00069d8-e907-4a18-a26e-834cd18fd911.png align="center")

You quickly realize, though, that `npm audit` will still display issues you've dismissed with Dependabot. Devs in your team installing new dependencies will still be ignoring the console warnings, and will probably not realize they're bringing in new vulnerabilities until the next Dependabot scan.

The summary of the security research you're doing with validating all issues before dismissing them is now hopefully stored in a separate documentation repository that people in your team can access - because if it's not, people in the future won't know how or why a specific issue has been dismissed as "risk is tolerable". How has the risk been assessed, and what does "tolerable" mean? That knowledge might be siloed.

Also, anyone who depends on your code is now seeing the same new vulnerability reports from Dependabot that you are getting. Every one of your users who cares about security is now re-doing your research, looking at your code and how you interact with the vulnerable library, before hopefully choosing to dismiss the issue from their repository. Some might just switch to a different library along the way. There's no way to inform the people that care about your security findings & resolutions.

And yeah, those automated fixes aren't always great... 😅

![](https://programmerhumor.io/wp-content/uploads/2022/11/programmerhumor-io-debugging-memes-javascript-memes-68768ffceb17351.png align="left")

## Not all issues are security advisories

Ok, you're now ready to raise some money and take your app to the next level! As part of the due diligence process, the lawyers ask you for a list of all your dependencies, including license info for each. This is when you realize you should have been paying attention to licenses all along. You scramble to figure out what SPDX and OSI are. You think your backend repositories are mostly safe since you don't distribute that code, but then you find out about [Network Protective licenses](https://www.fsf.org/bulletin/2021/fall/the-fundamentals-of-the-agplv3).

A tool to automate and enforce license compliance for your entire team would be nice!

* At this point, you've maybe found IBM's [audit-ci](https://github.com/IBM/audit-ci), which can help with allowlisting advisory IDs, but doesn't audit for any other types of issues except reported security vulnerabilities (same for [better-npm-audit](https://github.com/jeemok/better-npm-audit), or [npm-audit-resolver](https://github.com/naugtur/npm-audit-resolver));
    
* [license-checker](https://github.com/davglass/license-checker) is pretty popular, but only scans licenses and is [not maintained anymore](https://github.com/davglass/license-checker/issues/245);
    
* [ort](https://github.com/oss-review-toolkit/ort) scans for both vulnerabilities & licenses, but it's heavy and intimidating (Kotlin knowledge required), and doesn't include a way to dismiss alerts.
    

Meanwhile, you're stuck explaining to everyone on your team about this new license compliance thing they need to look out for.

Plus, have you recently tried getting `npm` to tell you which of your dependencies is *deprecated*? It only gives off warnings about that when you install new packages and you probably ignore it then. But you should care about deprecations too, right?

## Enter Sandworm 🪱

%[https://github.com/sandworm-hq/sandworm-audit] 

Sandworm Audit is a command-line tool designed to help with all of your auditing woes:

* It's free & open source!
    
* It lets you customize and own your security workflow
    
* It works with any modern JavaScript package manager
    
* It scans your project & dependencies for [vulnerabilities, license, and misc issues](https://docs.sandworm.dev/audit/issue-types)
    
* It supports [marking issues as resolved](https://docs.sandworm.dev/audit/resolving-issues)
    
* It supports [custom license policies](https://docs.sandworm.dev/audit/license-policies)
    
* It has [configurable fail conditions](https://docs.sandworm.dev/audit/fail-policies) for running in CI / GIT hooks
    
* Product of the day on Product Hunt!
    

[![Sandworm - Keep your JavaScript code secure and compliant with Sandworm | Product Hunt](https://api.producthunt.com/widgets/embed-image/v1/top-post-badge.svg?post_id=380800&theme=neutral&period=daily align="center")](https://www.producthunt.com/posts/sandworm?utm_source=badge-top-post-badge&utm_medium=badge&utm_souce=badge-sandworm)

Running a Sandworm audit will generate:

* A JSON report with all issue & license usage information;
    
* Easy to grok SVG dependency tree & treemap visualizations;
    
* A CSV list of all project dependencies & license info.
    

![](https://sandworm.dev/images/audit-terminal-output.gif align="left")

![](https://assets.sandworm.dev/showcase/treemap-and-tree.png align="left")

![](https://assets.sandworm.dev/showcase/csv-snip.png align="left")

## Build security & compliance into your app's core, today

With Sandworm, you're now able to take ownership of your security workflow, just like you always wanted:

* You add Sandworm to your CI to enforce security rules. You make a configuration file that you commit to your repo, and define a custom fail policy within it.
    
* You also add a permitted license policy to the configuration file, specifying which licenses are ok and which should trigger issues that need to be investigated. Everyone in the team sees what the rules are - you're all on the same page.
    
* You create a `resolved-issues.json` file in your repo, and tell your team to use `sandworm resolve` to dismiss issues that are not addressable by writing code. All resolutions are recorded in the JSON file and your git history, allowing anyone to historically track who resolved what, when, and why.
    
* Updates to the resolution file fit right into your existing code PR approval workflow.
    
* You share your resolution file with your users, to inform them of the issues you've investigated, and what they need to look out for.
    

## Try out Sandworm

We're a small team of long-time techies. The story in this article is our story, over the past five years, handling engineering for several JavaScript-powered startups, where we just wanted to get security right. We made Sandworm with ❤️ and ☕, in the hope that it will change all dev's minds about the importance of security practices, by providing them with the tools they always needed.

You can try out Sandworm in your app by running `npx @sandworm/audit@latest`, or by installing it globally with `npm i -g @sandworm/audit` and then running `sandworm audit` in your app's directory.

More about how Sandworm works in the [documentation](https://docs.sandworm.dev/). Questions, ideas, or feedback? Let me know in the comments! 👇 And if you like Sandworm, please star it on GitHub below to let us know you care about security:

%[https://github.com/sandworm-hq/sandworm-audit]