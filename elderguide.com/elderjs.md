# Elder.js: An Opinionated, SEO focused, Svelte Framework.

Elder.js is an opinionated static site generator and web framework built with SEO in mind. Supports SSR and Static Site Generation.

**Note:** Elder.js is currently in beta but approaching v1.0.0 quickly. (This site runs on it.)

<a class="github-button" href="https://github.com/elderjs/elderjs" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star elderjs/elderjs on GitHub">Star</a>

**Features:**

* Svelte on the server and the client. (Full SSR support or Static Export)
* Dead simple data flow.
* Fast static site generation.
* Fully customizable with hooks / plugins.
* Built with SEO in mind.
* Intuitive defaults.
* Easy partial hydration of Svelte components for tiny bundle/html sizes.


## Getting Started:

The quickest way to get started is to get started with the Elder.js template using [degit](https://github.com/Rich-Harris/degit):

Here is a demo of the template: [https://elderjs.netlify.app/](https://elderjs.netlify.app/)

**Step 1: Clone Template**

```bash
npx degit Elderjs/template elderjs-app
cd elderjs-app
```

**Step 2: Install Dependencies**

```bash
npm install # or just yarn
```

**Step 3: Start the Project**

```bash
npm start
```

Navigate to <a href="localhost:3000" rel="nofollow">http://localhost:3000</a>. You should see your app running.

### Developing using the Template:

For development, we recommend running two separate terminals. One for the server and the other for rollup.

**Terminal 1: Sever**

```bash
npm run dev:server # `npm start` above starts a server, but doesn't rebuild your Svelte components on change.
```

**Terminal 2: Rollup**

```bash
npm run dev:rollup # This rebuilds your svelte components on change.
```

Once you have these two terminals open, edit a component file in `src`, save it, and reload the page to see your changes.

### To Build/Serve HTML:

```bash
npm run build
```

Let the build finish.

```bash
npx sirv-cli public
```


## Routes

At the core of any site are it's "routes" or templates. 

In Elder.js a **route** is made up of 3 files that live in your route folder: `./src/routes/${routeName}/` folder.

They are:

1. A `route.js` file. This is where you define route details such as the route’s `permalink` function and `all.` function. 
1. A data.js file where you define what data the template should get.
1. A Svelte component to be used as a template matches the `${routeName}`. eg: `./src/routes/blog/Blog.svelte` (from here on out we refer to these specific Svelte components as "Svelte Templates")


### Route.js

`route.js` files are required to have an `all` function and a `permalink` function. 

Here is what a route for a website's homepage might look like it's simplest form.

```javascript
// ./src/routes/home/route.js
module.exports = {
  all: async () => [{ slug: '/' }], // an array of request objects.
  permalink: async ({ request, settings }) => request.slug, // a function that turns request objects into relative urls. 
};
```

Two things are happening here:

* `all()`: The all function returns an array of all of the "request" objects for a given route. Since this is the homepage, it returns an array with a single request object of `{slug: '/'}`.
* `permalink()`: The permalink function transforms request objects into permalinks. In this example, the "permalink" for `{slug: '/'}` is `/`. This makes sense because "/" is the relative url of the homepage of a site.

Together the `all` and `permalink` functions are used to complete map of all of the urls a specific route should handle.


### How Routing Differs from Express-like Frameworks

If you are familiar with `express` or `polka` then you are likely familiar with defining your routes as `/blog/:id/` where `:id` is a parameter that is passed in a route.

While this type of routing is familiar to many in the JS community, it has some distinct limitations when building a static site generator. Specifically:

1. When generating your static site, 'crawl' all of the links of a site to generate a complete map of urls.
1. Complex logic or regex is needed to have `/senior-living/:nursingHomeId/` and `/senior-living/:blogId/`.
1. It assumes that all parameters needed to generate a page can be determined by the URL.

### Inverted Routing

To solve the issues inherent with express-like routing, Elder.js uses what we refer to as an "inverted router" to build a map of all available pages.

How this works, is that for each `route` Elder.js:

1. Compiles all of the `request` objects by running the route's `all` function. 
1. The request objects are then passed to the `permalink` function to generate URLs.

Once it has gathered all of the permalinks, Elder.js now has a complete map of every page it should build or handle SSR for. 

This approach was chosen to give the user complete control of their URL structure instead of being bound by the limitations of being able to determine a route solely by a URL.



### all() Functions
Here is the function signature for a `route.js` all function:

```javascript
{
  all: async ({ settings, query, data}): Array<Object> => {
    // settings: this describes the Elder.js bootstrap settings.
    // query: an empty object that is usually populated on the 'bootstrap' hook with a database connection or api connection. This is sharable throughout all hooks and functions.
    // data: any data set on the 'bootstrap' hook.
    return Array<Object>;
  }
}
```


### permalink() Functions

Here is the function signature for a `route.js` permalink function:

```javascript
{
  permalink: ({ request, settings}): String => {
    // NOTE: permalink must be sync. Async is not supported.

    // request: this is the object received from the all() function. Generally we recommend passing a 'slug' parameter but you can use any naming you want.
    // settings: this describes the Elder.js bootstrap settings.
    return String;
  }
}
```




## Data.js Files

Whether you’re building are a personal blog or complex data driven SEO site, your `/routes/[route]/data.js` file is the recommend place to fetch (from a db, api, or other source) and prepare data to be consumed by your Svelte templates.

Each route can include an optional `data.js` file which return a single object. This `data` object will be passed into the route's Svelte template.

Data.js files receive the following parameters: `data`, `query`, `helpers`, `settings`, `errors`, `perf`, `allRequests`.

For a basic understanding of what can be done in a `data.js` file we encourage you to look at [data file for the Home template in the `elderjs-template` project](https://github.com/Elderjs/template/blob/master/src/routes/home/data.js).

### Database Connections, APIs, and External Data Sources

`data.js` files are designed to be the central place to fetch data for a route but the implementation details are very open ended and up to you. 

Just about anything you can do in Node.js you can do in a data.js file. 

That said, one of the most common things a data file will need to do is to connect to a remote data store. 

The recommended way of setting up that connection is [to populate the `query` object on the `bootstrap` hook](https://elderguide.com/tech/elderjs/#hook-example-1-bootstrap).

Using this pattern allows you to share a database connection across the entire lifecycle of your Elder.js site.


## Hooks: How to Customize Elder.js

Elder.js hooks are designed to be modular, sharable, and easily bundled in to [Elder.js plugins](https://github.com/Elderjs/plugins) for common use cases... while still giving developers of all skill levels an easy way to customize core page generation logic to their own needs.

For a full overview of the hooks available, you can reference the [hookInterface.ts](https://github.com/Elderjs/elderjs/blob/master/src/hookInterface/hookInterface.ts) or the hooks list below.

In short there is a hook at every major step of the page generation process from system bootstrap  (the `bootstrap` hook) all the way to writing html to your computer (on the `requestComplete` hook).

### Hook Interface: the `mutable` and `props` Arrays

For each hook there are two arrays defined. `props` and `mutable` this defines the "contract" that the hook interface implements.

In short, `props` represents the parameters that are available and `mutable` represents which of those parameters can be changed.

This structure was implemented to keep all side effects limited and predictable.

Under the hood, all items in the `props` array that aren't in the `mutable` array are passed as a [Proxy](https://javascript.info/proxy).

This allows Elder.js to enforce the [`hookInterface.ts`](https://github.com/Elderjs/elderjs/blob/master/src/hookInterface/hookInterface.ts) contract.

It also keeps hooks/plugins that are shared publicly from doing unexpected things.


### Hook Lifecycle

<img src="https://elderguide.com/images/elderjs-hook-lifescyle-v3.png" alt="Elder.js hook Lifecycle" style="max-width:100%; margin:1rem 0;" />


{{output_hook_list}}



### Hook Example 1: `bootstrap`

Here is the `bootstrap` hook from the `hookInterface.ts` file.

```javascript
// From hookInterface.ts
{{output_bootstrap_hook}}
```

This hook is executed after Elder.js has bootstrapped itself and lets users run arbitrary functions at that point too.

Internally Elder.js uses this hook to automatically any user defined helpers `./src/helpers/index.js` to the `helpers` prop that is available on other hooks, in Svelte templates, and data files.

Here is what a simple hook defined in your `./src/hook.js` file might look like if you wanted to add a database connection to the `query` object which is available every time a hook is called:


```javascript
// ./src/hooks.js
const db = require('../db');
module.exports = [
  {
    hook: 'bootstrap',
    name: 'addDbToQuery',
    description: 'Adds our db object to the query object',
    priority: 2,  // lower is more important. The system runs it’s hooks at priority of 1.
    run: async ({ query }) => {
      return {
        query: { ...query, db },
      };
    },
  },
};
```

**In plain english:**

`addDbToQuery` runs on the `bootstrap` hook and adds the `db` object as a key to the `query` object.

If you wanted to initialize a database connection and make it available on all hooks and in data.js files, this is how you'd do it.


### Where to Organize Hooks

**USER HOOKS:**

Hooks can be defined in 3 places in your project.

1. In your `./src/hooks.js` file. - *We recommend organizing site wide hooks in this file.*
1. In one of your `./routes/[routeName]/route.js` file. - *We recommend organizing route specific hooks here. (you'll still need to add logic to make sure it only runs on that route.)*
1. Within a plugin.

**SYSTEM HOOKS:**

Under the hood, all of the hooks Elder.js runs are defined in the [@elderjs/elderjs `./src/hooks.ts`](https://github.com/Elderjs/elderjs/blob/master/src/hooks.ts).

### Advanced: Customizing the Hook Interface

For power users we've included two hooks that can be used to customize the Hook Interface to your needs: `customizeHooks` and `modifyCustomProps`.

Let's say your team was using Elder Guide for SSR and wanted to add an arbitrary 'user' object that is available and mutable on all hooks.


**Step 1**: Register a hook in your hooks.js file to run on the 'customizeHooks' hook. In your specified 'run()' function adjust the `hookInterface` to modify the 'props' and 'mutable' arrays on all hooks to include 'user'. (Loop through them all and push 'user' into each array.)

**Step 2**: In the same run() function as in step 1, modify the `customProps` property to include the `user` object. (customProps.user)


**Step 3**: If for some reason you need to modify the `customProps` you set initially based on a request, you can modify them on the 'modifyCustomProps' hook.


**Notes**

* **Naming here is important, do not overwrite / shadow any of the props in Elder.ts**
* While it may be tempting to offer this functionality to plugins it is not available because plugins should be side effect free.
* If you are looking to store data or custom data points for your plugin, look at the `init()` function on the plugin definition as you can use that closure to store data that is plugin specific between hook invocations.
* At this point there is no way to add additional hooks to the system. In the future we may add the ability for plugins to define their own hooks which would shadow system hooks. If you think a hook is needed that you don't see, open a PR and please detail it's use case.



## Specifications and Config
Below are details on common specifications and config requirements. 

### Config: `elder.config.js`

By default Elder.js looks for an `elder.config.js` file in your project root and will import any settings there and merge them with the default.

Below is what the default configuration file looks like. This is automatically generated if an `elder.config.js` file is missing.

```javascript
module.exports = {{output_default_config}}
```

### Elder.js Expected file structure
Elder.js expects a specific file structure outlined below. This is why we recommend you start with the Elder.js template.

You can configure or rename your src/build folders in your `elder.conig.js`.

```plaintext
Project Root
| elder.config.js
| package.json
| tsconfig.json (typescript only)
| rollup.config.js
| ... (other common stuff, .gitignore, svelte.config.js... etc)
| -- src
| -- | -- build.js
| -- | -- server.js
| -- helpers
| -- | -- index.js
| -- | -- ...
| -- assets
| -- | -- items to be copied to public at build.
| -- layouts
| -- | -- Layout.svelte
| -- routes
| -- | -- [route] ('blog' in this example)
| -- | -- | -- Blog.svelte
| -- | -- | -- route.js
| -- | -- | -- data.js (optional)
| -- plugins
| -- | -- [plugin] ('elderjs-plugin-markdown' for example)
| -- | -- | -- index.js
| -- components
| -- | -- [component] ('Contact' in this example)
| -- | -- | -- Contact.svelte


Typescript Projects:
| -- build
| -- | ... copy of your compiled ts from the src folder here.
```





### Hook Specification

Hooks are the core of how to make site level customizations. Below is the default spec for a hook.

```javascript
module.exports = {{output_hook_schema}}
```

### Plugin Specification

Plugins are a bundle of hooks with their own closure scope and `init()` function. Below is the default specification for a hook.

```javascript
module.exports = {{output_plugin_schema}}
```

### Route Specification

Routes can be defined by plugins or by including a `./src/[routeName]/route.js` file.

```javascript
module.exports = {{output_route_schema}}
```



### Requirements for `name` and `description` fields
In various places such as on hooks, plugins, and stacks, you'll see that Elder.js requires `name` and `description` fields.

While this requirement may seem like an added development burden initially, it makes it extremely easy for you to public share your hooks or bundle them into plugins in a way others can use. (Plus your future self will thank you when you need to modify your code a year from now.)

These fields are also used to generate the pretty pretty printouts of how long each hook is adding to your page generation time that you see when you enable `debug.performance = true` in your `elder.config.js`.

<img src="https://elderguide.com/images/performance-debug-true.png" alt="Elder.js Performance Debug True" style="max-width:100%; margin: 1rem 0;" />

As a team we've found build times to be especially important when building 10k+ page sites as 100ms adds 16+ minutes to your build time. What gets measured gets managed... and we know faster deploys leads to deploying more often.

### Stacks: Predictable String Concatenation

In a few places you may see that Elder.js is using 'stacks.' These are just our internal way of processing strings that need to be concatenated. 

Here are the type defs:

```javascript
export type StackItem = {
  source: string;
  string: string;
  priority: number;
  name: string;
};

export type Stack = Array<StackItem>;
```

In short a stack is an array of objects that are concatenated together in order. Think of it like a queue. An item with a priority of 1 is the highest priority. An item with a priority of 100 is the lowest.

(Feedback please: Should we rename priority or 'order'? Is the 1-100 priority intuitive?)

Hooks can add items to the stack, when the stack is processed it is sorted in order of priority and then all strings are concatenated.



## Plugins

Plugins are prepackaged hooks and/or routes that can be used to add additional functionality to an `Elder.js` site.

Plugins also have the added bonus of having **their own isolated closure scope where they can store data between hooks invocations**.

To use a plugin, it must be registered in your `elder.config.js` and can be loaded from `./src/plugins/${pluginName}/index.js` or from the entry point to an npm package `./node_modules/${pluginName}/`

### Official Plugins:

* [Critical Path CSS](https://github.com/Elderjs/plugins/tree/master/packages/critical-path-css) Quickly and easily generate and include critical path css for your Elder.js website.
* [Sitemap](https://github.com/Elderjs/plugins/tree/master/packages/sitemap) Automatically generate the latest sitemap for your Elder.js website on build.
* Want to randomly preview a page from any route. (coming soon) (We've found this incredibly valuable when building sites with 15k+ pages of a specific route.)

### Other Plugin Ideas:

* Want to use Elder.js to read your markdown files? Perfect use case for a plugin. ([example implementation](https://github.com/Elderjs/template/blob/master/src/plugins/elderjs-plugin-markdown/index.js))
* Want to upload your statically generated files to S3? Perfect use case for a plugin. (See plugin example below)
* Read all images in the project, compress them, cache them locally, and make them available to templates.
* RSS feed plugin

### Writing Your Own Plugin

If you're looking to write you own plugin for Elder.js, [we've setup an easy template to clone](https://github.com/Elderjs/plugins/).

Here is the command you can use to clone it locally without all of the git history using degit.

```bash
npx degit Elderjs/plugin-template elderjs-plugin
cd elderjs-plugin
```

### Plugin Example 1: S3 Upload

Here is what a plugin looks like:

```javascript
const plugin: PluginOptions = {
  name: 'elder-plugin-upload-s3',
  description: 'Uploads html and/or data.json file to s3',
  init: (plugin) => {
    // console.log(plugin); => returns this plugin object.
    plugin.data = { test: true };
    // NOTE! any data added in any hook or in the init function will
    // be persisted for the entire lifecycle of elder.js
    // this means the closure persists between server.js loads AND between pages during build.
    // Temporary data can be stored on this object,
    // but it is up to the plugin to clean up after itself and manage it's own state.
    return plugin;
  },
  hooks: [
    {
      hook: 'requestComplete',
      name: 'uploadDataObjectToS3',
      description: 'Uploads a data.json file to s3',
      priority: 100, // we want it to be last
      run: async ({ data, settings, request, plugin }) => {
        // console.log(plugin.test) => true
        if (settings.build === true && settings.deploy === true) {
          if (plugin.config.dataBucket && plugin.config.dataBucket.length > 0) {
            let dest = `${request.permalink.replace(/^\/+/, '')}data.json`;
            if (plugin.config.deployId) {
              dest = `${plugin.config.deployId}/${dest}`;
            }
            await s3Helper.uploadToS3(dest, JSON.stringify(data), 'application/json', plugin.config.dataBucket);
          }
        }
      },
    },
    {
      hook: 'requestComplete',
      name: 'uploadHtmlToS3',
      description: 'Uploads a html file to s3 bucket.',
      priority: 100, // we want it to be last
      run: async ({ settings, request, html, plugin }) => {
        if (settings.build === true && settings.deploy === true) {
          if (plugin.config.dataBucket && plugin.config.htmlBucket.length > 0) {
            let dest = `${request.permalink.replace(/^\/+/, '')}index.html`;
            if (plugin.config.deployId) {
              dest = `${plugin.config.deployId}/${dest}`;
            }

            await s3Helper.uploadToS3(dest, html, 'text/html', plugin.config.htmlBucket);
          }
        }
      },
    },
  ],
  config: {
    dataBucket: process.env.AWS_S3_DATA_BUCKET || '',
    htmlBucket: process.env.AWS_S3_BUCKET || '',
    deployId: process.env.DEPLOY_ID || false,
  },
};
```

This plugin registers function executions on two hooks, the `dataComplete` hook and the `requestComplete` hook. In each, it uploads the data or html to the s3 bucket specified in the user's `elder.config.js`.





## Svelte Templates and Svelte Components
Within Elder.js there is a subtle distinction between what we refer to as **Svelte components** and **Svelte templates**.

As single **Svelte Template** is defined for each route. (eg: `./src/routes/blog/Blog.svelte`)

Where **Svelte components** (eg: `./src/components/Widget/Widget.svelte`) are called from within **Svelte Templates**.

On a practical level, the only difference is how they are bundled by Rollup and that we only expect to see the `hydrate-client` prop within **Svelte templates**.

The technical reason for this is that **Svelte templates** are the backbone of how Elder.js does server side rendering (SSR) and hence don't have a client side bundle produced for them.






### Svelte Templates and Partial Hydration

Elder.js uses SSR Svelte components for it's templating. All of the magic you usually get with SSR Svelte templates is handled here. This includes `<svelte:head>`, `<style></style>`, `<script></script>` and more.

Each route folder is expected to have a Svelte component in it's folder that it will use as it's template. (`./src/routes/RouteName/RouteName.svelte` -- Notice the capitalization). Within these templates you can use the syntax below syntax on components and provided you are using the Elder.js rollup file and the `partialHydration` preprocessor. (You're probably best just copying it from the [`elderjs-template`](https://github.com/Elderjs/plugin-template))

```
// within any Svelte template
 <Something hydrate-client={{somethingCool: true}} />
```

What this will do is setup a new Svelte root component in the produced HTML and set the in initial props to `{somethingCool: true}`.

There are two differences between partial hydration and the way most frameworks handle hydration:

1. With partial hydration you're only sending the data needed to make the respective components interactive instead of data for the whole page.
1. With partial hydration you end up with multiple root components instead of one main component root. (Don't worry, you can still use Svelte stores allow them to easily share state across them.)

The end result is generally smaller bundle/page sizes and less work for the main thread. is that we're only hydrating what is needed by the client instead of all of the data to build the page.

### How partial hydration works under the covers.

On the homepage of elderguide.com we use the following code to hydrate the autocomplete component:

```svelte
// within elderguide.com’s Home.svelte
 <HomeAutoComplete hydrate-client={{ nh_count: data.nh_count }} />
```

You can do the same for a component without any props by using:

```svelte
  <NoPropsHere hydrate-client={{}} />
```

At a high level what is happening is that when the Svelte template components are compiled on the server, we've included a preprocessor that causes the Svelte compiler to instead render a div with a few specific elements and the prop passed into `hydrate-client` is simply JSON.stringified.

Later when we go to render these templates, we look for the removed components, generate the server rendered version and include the client component in the generated JS with the props that were given in `hydrate-client`.

If you are curious the files to look at are: `partialHydration.ts` and `svelteComponent.ts.`

The important thing to note is that still use Svelte variables in `hydrate-client` as long as they can be processed by `JSON.stringify`.



## Why We Built Elder.js

When we set out to build elderguide.com we tested 6 different static site generators (Gatsby, Next.js, Nuxt.js, 11ty, Sapper and Hydrogen.js) and ultimately realized there wasn’t a solution that ticked all of our boxes.

On our journey we had 3 major realizations:

1. The only SSG to support Svelte was Sapper the other most mature JS alternatives (Gatsby, Next.js, Nuxt.js) are all tightly coupled with React/Vue. This was a bummer as we loved the amazing developer experience Svelte offers.
1. Most SSGs are built for either simple sites/blogs or for full scale "app frameworks" that have added an 'export' process added as an after thought.
1. Client side routing adds a huge amount of complexity (and bundle size) to initial loads for very little SEO benefit. If you aren’t building an App, why would we want to fully hydrate our JS framework just for faster routing? Browsers a great at routing… we should only be hydrating things that need to be hydrated.

After our testing we decided to go with Sapper but ultimately hit some roadblocks.

This led us to the decision of rewriting all of our Svelte components to React/Vue or finding a different SSG.

In an afternoon of frustration, we whipped up a very rudimentary SSG with a mindbogglingly complex and error prone process of adding Svelte components… but hey, it worked. #productionready

Over the following 6 months as we pushed the site towards production, we used this SSG as our playground to explore the best way to build an SEO focused, data driven, statically exported sites.

We launched to production with a mix of `lit-html` with a few helper functions to hydrate/ssr Svelte components mixed in.

While we had always planned on refactoring once we hit production, a moment of genius from [@kev](https://github.com/kevmodrome) in the Svelte discord prompted a major breakthrough that allowed us to use Svelte 100% for templating and still get partial hydration even thought Svelte doesn’t support it.

Excited we completely completely rewrote our system to take advantage of this breakthrough and add in easy plugin/customization support so anyone could use it to build sites.




## Automatic Behavior
Elder.js does quite a bit of automatic behavior much of which is based on using the standard filesystem.

Instead of burying this magic, things that happen automagically are logged to the console unless you set `debug.automagic = false` in your `elder.config.js` file. That said, here are some of the things that happen automatically:

* All exports from `./src/helpers/index.js` are imported and added to the `helpers` object which is available in the `./routes/[routeName]/data.js` file and all hooks.
* All hooks in your `./src/hooks.js` file, `./routes/[routeName]/route.js` files, and plugins are imported in validated.
* When Elder.js detects a `./tsconfig.json` in your project root we check it for for a build folder. If it exists and you haven't defined it in your `elder.config.js` it will be set automatically.
* All optional variables in your routes file will be set automatically if Elder.js detects that the files needed are present.




## Default Helpers

* Permalink resolver: `helpers.permalink[routeName]()`. Simply pass in a request object and it'll resolve the permalink.
* `link`in templates. TODO.



## Elder.js Exports:

### `server`
Elder.js was built as a static site generator we offer a built in server that is pretty fast past the intial load.

We recommend you use this server as a previewer with `express` or `polka`.

You can see how this is done in the `elderjs-template` in the server.ts.

### `build`
The `build` function exported from Elder.js uses Node's cluster module to use multiple processes to build your site.

In short, here is how `build` work at a high level:

The main process runs through bootstrap and collects `allRequests`. It then spins up workers and divides the requests evenly across these workers. It then waits for the workers to complete, runs any hooks for after build and completes.

There are two notable config options: `numberOfWorkers` and `shuffleRequests` which you can read about in the config section.


### `partialHydration`

This is the rollup preprocessor that Elder.js expects to be used when rollup is bundling your Svelte templates and components.






## FAQ



### Why can't I use Svelte templates for data fetching/manipulation?


If you’re familiar with Sapper you may be accustom to storing you data fetching and manipulation logic in your Svelte files.


Initially when designing Elder.js we experimented with this approach but found that a separate data.js file which returned a single data object ready to be consumed by you svelte template was a cleaner separation of concerns.


Data files for fetching, manipulating, and returning data. Svelte for handling data presentation, interactivity, and reactivity.


Note: If you really want to do data fetching and manipulation in your templates nothing is stopping you.




### How Does Elder.js Differ From Sapper?

Sapper is a full stack Svelte framework that does server site rendering and supports static exporting of your site.

While we found Sapper to be a nice solution for small sites, when we tried to use it to build out elderguide.com we hit some major roadblocks:

1. V1 of our internal test site had about ~500 pages. Sapper was talking about 15 minutes to build our site and we had worked hard to improve that speed. Since we knew our site would have over 100k pages in the coming years we knew we needed a different solution.
1. Sapper uses a file encoded routing system. This means that /blog/[id]/ becomes /blog/:id/.  While this type of routing is common in JS frameworks there are several SEO reasons you may want content of different types to live within the same subfolder. (In our specific use case we wanted /senior-living/[content]/, /senior-living/[facilities]/, and /senior-living/[experts]/ to all live under /senior-living/. When you're planning on building a 100k page site over 5 years, early decisions matter. Especially for SEO as each permalink change adds a bit of 'drag.')
1. We found Sapper’s data flow hard to reason about for data driven and complex sites.

For data fetching Sapper uses a `preload` function to fetch a page’s initial props and build the page. This is done to enable client side routing and full client side hydration as this preload function is designed to run on both the client and the server.

This is a pretty cool feature for Apps but for a static SEO focused site it was overkill and cause huge problems with our dataflow when paired with the chosen routing system.

Elder.js solves these roadblocks.

1. Build times on our ~20k page site are routinely less than 10 minutes on a modest VM. In our analysis over 55% of our build time is spent waiting on the database server so you may see much faster builds. :)
1. You can setup the routes however you need. The only requirement is that you don’t have two routes with the same url. :)  (We’ll tell you if is happens)
1. Every route defined has a single `data.js` where you can do your database queries, read markdown files, and do whatever magic you need to prepare your data for display.


### Adding Database Access

If your project is going to be querying it’s data from a database we recommend using the bootstrap hook and adding a connection to your database on the “query” property. [See this hook example above](https://elderguide.com/tech/elderjs/#hook-example-1-bootstrap).




