---
title: "Security Alert: Don't `npm install https`"
datePublished: Thu Jun 15 2023 13:38:40 GMT+0000 (Coordinated Universal Time)
cuid: clix6sixf000e09ml3lj5fjda
slug: security-alert-dont-npm-install-https
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686833678249/dfa17f21-5147-4677-908f-4a5c3967b4fa.png
tags: https, security, nodejs, npm

---

The Node.js `https` module is a built-in module that allows you to make secure HTTPS (Hypertext Transfer Protocol Secure) requests to servers. It provides a way to communicate securely over the internet by encrypting the data transmitted between the client and the server using SSL/TLS protocols.

Here's a very simple usage example of the `https` module in Node.js:

```javascript
const https = require('https');

const options = {
  hostname: 'api.example.com',
  path: '/data',
  method: 'GET'
};

const req = https.request(options, (res) => {
  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('end', () => {
    console.log(data);
  });
});

req.on('error', (error) => {
  console.error(error);
});

req.end();
```

A package called `https`, however, also exists on npm:

%[https://www.npmjs.com/package/https] 

The `https` package has last been published 8 years ago. Looking at the package contents, we can see that it contains just the manifest file, and no actual code:

```json
{
  "name": "https",
  "version": "1.0.0",
  "description": "https mediation",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "https",
    "mediation"
  ],
  "author": "hardus van der berg <hardus@sunfork.com> (http://www.sunfork.com)",
  "license": "ISC"
}
```

It looks like a simple, useless package until we see that [over 1600 other packages depend on it](https://www.npmjs.com/browse/depended/https) directly, and many more depend on it indirectly, without even knowing. This is probably the result of maintainers not being aware that `https` is internal to Node, and thinking they need to install it - it's also been asked on StackOverflow [multiple](https://stackoverflow.com/questions/12028046/npm-install-https) [times](https://stackoverflow.com/questions/27376713/nodejs-npm-install-https-not-working).

Node's `require` will [load built-in modules first](https://nodejs.org/api/modules.html#modules_all_together), so even if code would be included in a new version of the npm `https` package, it would never run via a simple `require('https')`. **The danger of having** `https` **in your list of dependencies rather comes from the possibility of the package being reactivated with a new version that contains install scripts.** We've written before about how install scripts can be dangerous:

%[https://blog.sandworm.dev/dissecting-npm-malware-five-packages-and-their-evil-install-scripts] 

The `https` package currently gets more than 500,000 downloads per week. At any moment in the future, publishing a `1.0.1` version with malicious install scripts would enable potential attackers to access a great number of development and build machines, to steal information or install arbitrary software.

The [Sandworm](https://sandworm.dev/) team has reached out to the package maintainers and authors that currently rely on `https` to suggest they remove it from their dependencies.

---

**If you enjoyed this article, have a look at Sandworm to keep your JavaScript project safe!** 👇

%[https://github.com/sandworm-hq/sandworm-audit] 

> [***Sandworm Audit***](https://github.com/sandworm-hq/sandworm-audit) ***is the open-source*** `npm audit` *that doesn’t suck: it checks for multiple types of issues, like vulnerabilities or license compliance, it outputs SVG charts and CSVs, it can mark issues as resolved, and you can also run it in your CI to enforce security rules.* [***Check the docs***](https://docs.sandworm.dev/) *and* `npx @sandworm/audit@latest` *in your JavaScript app’s root to try it out 🪱.*
> 
> ![](https://cdn-images-1.medium.com/max/2120/1*JvTQJzapGNSSCKWQi89hCA.gif align="left")