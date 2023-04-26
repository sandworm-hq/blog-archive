---
title: "Running Eleventy Serverless On AWS Lambda@Edge"
datePublished: Fri Mar 10 2023 10:12:56 GMT+0000 (Coordinated Universal Time)
cuid: clf2dqb9z000509mgh5wgf7t9
slug: running-eleventy-serverless-on-aws-lambdaedge
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678442188717/5e27892c-e6f2-4f47-aaeb-bc88e3cebdbe.png
tags: lambda, netlify, static-website, eleventy, lambdaedge

---

[Eleventy](https://11ty.dev) is great. It’s a static site generator written in JavaScript, for “*Fast Builds and even Faster Web Sites.*” It’s 10 to 20 times faster than the alternatives, like [Gatsby](https://www.gatsbyjs.com/) or [Next.js](https://nextjs.org/). You get all of your content statically rendered and ready to be CDN-delivered. You needn’t worry about server-side rendering to get those pretty [social share unfurls](https://socialsharepreview.com/). And, if you have a large data set, that’s great — Eleventy can [generate tens of thousands of pages](https://github.com/11ty/eleventy/issues/2226) with no issues.

### But what if you have a HUGE data set?

When building Sandworm’s [open-source security & license compliance audits for JavaScript packages](https://sandworm.dev/), we wanted to generate a catalog of beautiful report visualizations for every library in the npm registry. That is, for every version of every library in the registry. We soon found out — that’s more than **30 million package versions**. Good luck generating, uploading, and keeping that amount of HTML pages up to date in a decent amount of time, right?

We looked at reducing our data set to just the most popular packages. We looked at implementing partial builds, where stale report pages would get continuously generated and uploaded.

But the solution we ended up implementing was [**Eleventy Serverless**](https://www.11ty.dev/docs/plugins/serverless/), a plugin that runs one or more template files *at request time* to generate dynamic pages. So instead of going through the entire set of pages at build time, this plugin allows us to separate “regular” content pages rendered at build from “dynamic” pages rendered on demand. We can then simply generate and upload static content (like the homepage, about page, etc.) in the CI, and then deploy some code to a compute provider that will generate an npm package page when a user navigates to a specific URL. Great!

**Except: Eleventy Serverless is built to work out-of-the-box with** [**Netlify Functions**](https://www.netlify.com/products/functions/)**, and we’re running on AWS.**

The good news is that you **can** get Eleventy Serverless to run in AWS Lambdas. Even better, you can get it to run in [**Lambda@Edge**](https://aws.amazon.com/lambda/edge/), which runs your code globally at AWS locations close to your users so that you can deliver full-featured, customized content with high performance and low latency.

%[https://github.com/gabidobo/11ty-lambda-edge-demo] 

### Setting up Eleventy

First things first: let’s get Eleventy running the local build. We start by installing it:

```bash
npm i @11ty/eleventy --dev
```

Then, let’s create the simplest template for our static Eleventy page. We’ll write it using [Liquid](https://shopify.github.io/liquid/), but since it’s so simple, it won’t take advantage of any useful templating tags for now. Let’s call it `index.liquid`:

```xml
<h1>Hello</h1>
```

That’s it, we’re ready to build and serve! Run:

```bash
npx @11ty/eleventy --serve
```

```bash
[11ty] Writing _site/index.html from ./src/index.liquid
[11ty] Serverless: 3 files bundled to ./serverless/edge.
[11ty] Wrote 1 file in 0.12 seconds (v2.0.0)
[11ty] Watching…
[11ty] Server at http://localhost:8080/
```

Visit [http://localhost:8080/](http://localhost:8080/) in your browser at this point, and you should see the “Hello” heading we’ve created above. Neat!

### Setting up the Eleventy Serverless plugin

The Serverless plugin is bundled with Eleventy and doesn’t require you to npm install anything. We do need to configure it, though. To do that, we need to create an Eleventy config file:

```js
// .eleventy.js
const { EleventyServerlessBundlerPlugin } = require("@11ty/eleventy");

module.exports = function(eleventyConfig) {
  eleventyConfig.addPlugin(EleventyServerlessBundlerPlugin, {
    name: "edge",
    functionsDir: "./serverless/",
    redirects: false,
  });

  return {
    dir: {
      input: 'src',
    },
  };
};
```

Let’s break the plugin configuration down:

* Each plugin’s unique `name` will determine the build output directory name and will be required when assigning permalinks. You can also instantiate multiple plugins to have different functions handle different pages.
    
* The `functionsDir` allows you to specify the path to the build output-dir; in our case, the plugin will generate files in the `./serverless/edge` directory relative to the app root.
    
* `redirects` configure how Netlify redirects should be handled — since we’re not running on Netlify, we set this to false to skip generating a `netlify.toml` file.
    
* Lastly, in the configuration object we return to Eleventy, we specify an input dir for our content to keep things tidy. We’ll also go ahead and move the `index.liquid` file we created earlier in the src directory.
    

Next, let's build again by running `npx @11ty/eleventy`, and investigate what gets output under `./serverless/edge`. You should see the following:

* Several js and JSON files starting with `eleventy-`. Some are configuration files, and some are built to inform the Netlify bundler of function dependencies — we won’t need those for Lambda.
    
* `eleventy.config.js`, a copy of the main configuration file in the app root.
    
* The `src` directory with the `index.liquid` template.
    
* An `index.js` file with the actual serverless handler code that we’ll update and deploy to Lambda.
    

Let’s gitignore the build artifacts that we don’t want in our repo and only keep the `index.js` file for now. Add this to your `.gitignore` file:

```bash
serverless/edge/**
!serverless/edge/index.js
```

Good, we’re now ready to create our first dynamically generated page. Let’s make another simple Liquid file for it under `src/edge.liquid`:

```html
---
permalink:
  edge: /hello/
---
<h1>Hello@Edge</h1>
```

You’ll notice that for this file, we’ve added some [front-matter data](https://www.11ty.dev/docs/data-frontmatter/) to the liquid template. Specifically, we’ve defined a permalink for our page to respond to when running under the edge plugin. Eleventy won’t generate an `edge.html` page when building — this page will only be generated by invoking the serverless handler code.

### Making things Lambda-compatible

Let’s now look at what’s going on with `serverless/edge/index.js`. This is only generated with the initial build, so we’re free to modify it — and we’ll need to in order to support Lambda@Edge.

* First, we can remove the `require("./eleventy-bundler-modules.js")`, as that’s only needed for the Netlify bundle process;
    
* Next, we’ll need to get a reference to the current request path and query, as Eleventy needs that info to know what content to generate. With Netlify, you get these via `event.rawUrl`, `event.multiValueQueryString`, and `event.queryStringParameters`. With Lambda@Edge, we’ll be getting events generated by CloudFront on origin requests — see [an example in the AWS docs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html#example-origin-request). We’ll also use `querystring` to handle parsing the query string. Let’s update the code to this:
    

```js
const { request } = event.Records[0].cf;
const path = `${request.uri}${request.uri.endsWith("/") ? "" : "/"}`;
const query = querystring.parse(request.querystring);

let elev = new EleventyServerless("edge", {
  path,
  query,
  functionsDir: "./",
});
```

* Finally, we need to update the handler’s returned objects to match the format expected by Lambda@Edge. Update the success & error responses status and headers to:
    

```json
{
  status: "200",
  headers: {
    "cache-control": [
      {
        key: "Cache-Control",
        value: "max-age=0",
      },
    ],
    "content-type": [
      {
        key: "Content-Type",
        value: "text/html; charset=UTF-8",
      },
    ],
  },
  body: ...
}
```

* We’ve also added a `Cache-Control` header to configure how CloudFront caches the returned results. We can get more thoughtful about this when moving to production, but for now, we’ll go with no caching.
    

One last thing: we’ll want to separate build dependencies from edge handling dependencies, so let’s create a separate `package.json` file in `serverless/edge`, and install `@11ty/edge` as a prod dependency. As our edge function grows, we’ll add more things here, like database clients.

---

> And as you add more dependencies, it’s time to also build security and compliance into your app early. [Sandworm Audit](https://github.com/sandworm-hq/sandworm-audit) is the open-source `npm audit` that doesn’t suck: it checks for multiple types of issues, like vulnerabilities or license compliance, it outputs SVG charts and CSVs, and you can also run it in your CI to enforce security rules. [Check the docs](https://docs.sandworm.dev/) and `npx @sandworm/audit` in your JavaScript app’s root to try it out 🪱.

![Sandworm Audit](https://cdn-images-1.medium.com/max/2120/1*JvTQJzapGNSSCKWQi89hCA.gif align="left")

---

Here’s our full handler code, for reference:

%[https://gist.github.com/gabidobo/c7ead7f55caaabf2ae20e8b851a18260] 

### Testing it out locally

Good, let’s test this out locally before we deploy! It should be pretty easy to simulate sending an event to our handler function. Let’s create a simple `test.js` file:

```js
const { handler } = require('.');

(async () => {
  const response = await handler({Records: [{cf: {request: {uri: "/hello/", querystring: ""}}}]});

  console.log(response);
})();
```

Running `node test.js` in the console, you should see:

```json
{
  status: '200',
  headers: { 'cache-control': [ [Object] ], 'content-type': [ [Object] ] },
  body: '<h1>Hello</h1>'
}
```

Take a moment to celebrate! You’ve just triggered your first Eleventy build in a serverless function. 🎊

### Deploying to AWS

Things look good — it’s now time to deploy this to AWS. To handle the deployment, we’ll be using Serverless. No, not the Eleventy Serverless plugin, but [Serverless](https://www.serverless.com/), the “*zero-friction development tooling for auto-scaling apps on AWS Lambda*” command-line tool. If you don’t have it installed, `run npm install -g serverless`. Then create a `serverless/edge/serverless.yml` file to configure the deployment:

%[https://gist.github.com/gabidobo/baa8e0c0f4132c53271a753b952688c7] 

* This will instantiate a CloudFront distribution connected to the bucket you specified under `events>cloudfront>origin`. Any calls to URLs matching the pathPattern will be forwarded to the serverless handler instead of being routed to the bucket.
    
* Fun fact: Lambda@Edge functions log console output to their regional CloudWatch. That is, if a user in Germany accesses your pages via the edge at `eu-frankfurt-1`, you’ll see logs for that specific run under the `eu-frankfurt-1` region and nowhere else. In the yml config, we make sure to give our function proper permissions to write log groups anywhere.
    
* We should also add an exception for the config file to `.gitignore` — we want this in the repo.
    
* If you already have a CloudFront distribution that you want to connect to your new serverless function, check out the [serverless-lambda-edge-pre-existing-cloudfront](https://www.serverless.com/plugins/serverless-lambda-edge-pre-existing-cloudfront) plugin.
    

We’re ready to deploy! Make sure you export AWS credentials for an IAM user with proper permissions for deploying the entire stack. When moving to production, for security purposes, you should create a dedicated user with the minimal set of permissions required — however, I haven’t been able to find a comprehensive list of such permissions, so this will likely be a tedious trial-and-error process of figuring them out by trying deploys and seeing what fails. While still in development, an admin user might be easier to use.

Run `sls deploy --stage prod` to deploy. If all goes well, in a couple of minutes, you should see the URL to your new CloudFront distribution! Your settings will need to propagate globally though, so it might take a few more minutes for everything to be ready. You can check the current status of your distribution under the AWS console dashboard. Once it’s done deploying, navigating to `CF_URL/hello` in a browser should display our “*Hello@Edge*” html header from the edge.liquid template. We did it! 🙌

### Bonus: making it dynamic

Now let’s quickly make our serverless function actually do something async. Let’s have it accept a URL parameter that’s the name of a Pokémon, and respond with an image of said cute beast. We’ll use [https://pokeapi.co/](https://pokeapi.co/) to get the image.

We could do the async work outside of eleventy, and then inject some global data like this:

```js
const eleventy = new EleventyServerless('serverless', {
  path,
  query,
  functionsDir: './',
  config: (config) => {
    config.addGlobalData('data', yourData);
  },
});
```

Or, better yet, starting with Eleventy 2.0.0, we can use [async filters](https://www.11ty.dev/docs/filters/#asynchronous-filters). Let’s first update our edge.liquid template to include the new HTML we want:

```plaintext
---
permalink:
  edge:
    - /hello/
    - /hello/:name/
---
<h1>Hello@Edge \{\{ eleventy.serverless.path.name }}</h1>
{% if eleventy.serverless.path.name %}
  <img src="\{\{ eleventy.serverless.path.name | escape | pokeimage }}" />
{% endif %}
```

* We’ve added a new `permalink` that includes a name path parameter. That will become available in the data cascade as [`eleventy.serverless.path.name`](http://eleventy.serverless.path.name).
    
* We’re transforming this `name` param via two filters: `escape` and `pokeimage`. Remember, user input should be treated as potentially malicious 😉.
    
* We need to define our `pokeimage` filter. This is where the async magic happens. Add this to your `.eleventy.js` file:
    

```js
eleventyConfig.addAsyncFilter("pokeimage", async function(name) {
  const results = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`);
  const json = await results.json();

  return json.sprites.front_default;
});
```

We’re relying on node’s built-in `fetch` API here — it’s a good thing we’ve set `runtime: nodejs18.x` in our `serverless.yml` file.

Let’s update our `test.js` file to query the `/hello/ditto/` URL, and run `node test.js` again. In the console output, you should now see:

```json
{
  status: '200',
  headers: { 'cache-control': [ [Object] ], 'content-type': [ [Object] ] },
  body: '<h1>Hello@Edge ditto</h1>\n' +
    '\n' +
    '  <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/132.png" />\n'
}
```

One last `sls deploy --stage prod` to get this deployed, and done! You’ve mastered setting up Eleventy Serverless on Lambda@Edge.

---

All of [Sandworm’s npm package report pages](https://sandworm.dev/npm/package/npm/4.0.2/) are generated using Eleventy Serverless and Lambda@Edge. Sandworm is about JavaScript security and compliance.

* Audit your dependencies and licenses using [Sandworm Audit](https://docs.sandworm.dev/audit).
    
* Sandbox dependencies and monitor your running app using [Sandworm Guard](https://docs.sandworm.dev/guard).
    

![https://sandworm.dev](https://cdn-images-1.medium.com/max/2000/1*t3IsqAZ5SLjQ_qYVc6zEEw.png align="center")

### **%%\[promo\]**