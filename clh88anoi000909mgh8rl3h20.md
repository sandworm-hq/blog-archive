---
title: "The Npm Packages That Troll You"
seoDescription: "Like any powerful tool, npm install scripts can be abused. Let's look at six non-malicious joke packages, to see how they troll or surprise their users."
datePublished: Wed May 03 2023 21:46:49 GMT+0000 (Coordinated Universal Time)
cuid: clh88anoi000909mgh8rl3h20
slug: the-npm-packages-that-troll-you
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683122666438/17a21a7c-c0ae-44e2-8bf1-2476aa670023.webp
tags: javascript, security, npm, trolling

---

Npm install scripts are a powerful tool for developers who want to automate tasks related to installing and configuring their packages. They can be used to perform tasks like setting up environment variables, running tests, and building production-ready code.

But like any powerful tool, npm install scripts can also be abused. One of the biggest concerns is that malicious packages can include scripts that execute automatically during installation, potentially compromising the security of the user's system. This can range from simple pranks or trolling to [more serious malware that can steal sensitive information or take control of the user's machine](https://blog.sandworm.dev/dissecting-npm-malware-five-packages-and-their-evil-install-scripts).

For this article, though, let's ignore the malware and only look at six non-malicious joke packages, to see how they exploit install scripts to troll or surprise their users.

## The Classic Rickroll

%[https://www.npmjs.com/package/ember-data-react] 

```json
"scripts": {
  "open": "xdg-open 'https://www.youtube.com/watch?v=dQw4w9WgXcQ'",
  "preinstall": "npm run open",
  "test": "npm run open"
}
```

The `embed-data-react` package makes use of `xdg-open`, a command-line utility that is used to open files or URLs using the default application for the file type or URL scheme on a Linux-based system. This will not work on Mac or Windows machines, since they don't have the `xdg-open` command.

On Linux, the preinstall script opens the classic rickroll video from YouTube:

%[https://www.youtube.com/watch?v=dQw4w9WgXcQ] 

%[https://www.npmjs.com/package/peekatchuysharmlesspackage] 

```json
"scripts": {
  "preinstall": "touch ../../evil",
  "postinstall": "brave https://www.youtube.com/watch?v=dQw4w9WgXcQ?autplay=1 && cat /etc/passwd > evil",
  "test": "echo \"Error: no test specified\" && exit 1"
}
```

`peekatchuysharmlesspackage` doesn't try to hide that it's only a demo of how malicious npm packages can steal data and mess with developer's machines. If you do install it though, apart from simulating some other "evil" actions, it will try to use the Brave browser to open the same Rickroll video as before.

## The Insult To Injury

%[https://www.npmjs.com/package/inferium] 

```json
"scripts": {
  "preinstall": "for i in {1..10}; do firefox https://www.youtube.com/watch?v=48rz8udZBmQ & firefox https://www.youtube.com/watch?v=FavUpD_IjVY & done &"
}
```

If you have `firefox` installed on Linux and you `npm i inferium`, the preinstall script will open 20 new Firefox tabs with the following videos playing:

%[https://www.youtube.com/watch?v=48rz8udZBmQ] 

%[https://www.youtube.com/watch?v=FavUpD_IjVY] 

## The Surprise Bernie

%[https://www.npmjs.com/package/bernie2020] 

Although it mentions "mining cryptocurrencies" in the description, the `bernie2020` package does nothing but open a bundled jpeg of Bernie via the `postinstall` hook. Note that the `open` command will only work on macOS.

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "postinstall": "open Bernie_Sanders.jpg"
},
```

![The bundled photo of Bernie](https://cdn.hashnode.com/res/hashnode/image/upload/v1683127927516/87cda035-35d2-40bc-9331-3ffc6844fd83.jpeg align="center")

## The Practical Jokers

%[https://www.npmjs.com/package/charlie-irl] 

%[https://www.npmjs.com/package/thanks-austin] 

```json
"scripts": {
  "postinstall": "open -a \"Google Chrome\" charlie-irl.png"
}
```

User [austinkelleher](https://www.npmjs.com/~austinkelleher) seems to have played a practical joke on user [xdumaine](https://www.npmjs.com/~xdumaine) by publishing a `charlie-irl` package to the npm registry that, upon installation, uses the `open` command to load a bundled image in Chrome. User [xdumaine](https://www.npmjs.com/~xdumaine) then followed up with a similar package named `thanks-austin`.

## **Any** `npm install` Comes With A Risk

In this article, we've only looked at some benign joke packages, but install scripts are no laughing matter. **Around 700k package versions in the npm registry currently declare install scripts, and could thus execute code that compromises the security of your dev machine, or your CI build.** More are added every day.

To mitigate this risk, it's important for JavaScript developers to carefully review the install scripts of the packages they use and to only install packages from trusted sources. Developers should also consider using tools like [`sandworm audit`](https://blog.sandworm.dev/the-better-npm-audit) to [**identify any security vulnerabilities in their dependencies**](https://blog.sandworm.dev/the-better-npm-audit) and stay up to date with security updates.

**If you enjoyed this article, have a look at Sandworm to keep your JavaScript project safe!** 👇

%[https://github.com/sandworm-hq/sandworm-audit] 

> [***Sandworm Audit***](https://github.com/sandworm-hq/sandworm-audit) ***is the open-source*** `npm audit` *that doesn’t suck: it checks for multiple types of issues, like vulnerabilities or license compliance, it outputs SVG charts and CSVs, it can mark issues as resolved, and you can also run it in your CI to enforce security rules.* [***Check the docs***](https://docs.sandworm.dev/) *and* `npx @sandworm/audit@latest` *in your JavaScript app’s root to try it out 🪱.*
> 
> ![](https://cdn-images-1.medium.com/max/2120/1*JvTQJzapGNSSCKWQi89hCA.gif align="left")