# Getting Started

Before getting started, make sure you are using a recent version of [Node](http://nodejs.org/). I recommend using at least the most current LTS (long-term support) version. The configuration of the book has been written Node 6 features in mind. You should have `node` and `npm` commands available at your terminal. [Yarn](https://yarnpkg.com/) is a good alternative to npm and will work for the tutorial as well.

The completed configuration is available at [GitHub](https://github.com/survivejs-demos/webpack-demo). If you are unsure of something, refer there.

T> It is possible to get a more controlled environment by using a solution such as [Docker](https://www.docker.com/), [Vagrant](https://www.vagrantup.com/) or [nvm](https://www.npmjs.com/package/nvm). Vagrant comes with a performance penalty as it relies on a virtual machine. Vagrant is valuable in a team: each developer will have the same environment, usually close to production.

W> If you are using an older version than Node 6, you may either have to adapt the code or process your webpack configuration through Babel as discussed in the *Loading JavaScript* chapter.

## Setting Up the Project

To get a starting point, we should create a directory for our project and set up a *package.json* there. npm uses that to manage project dependencies. Here are the basic commands:

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y # -y generates *package.json*, skip for more control
```

You can tweak the generated *package.json* manually to make further changes to it. We’ll be doing changes through *npm* tool, but manual tweaks are acceptable. The official documentation explains [package.json options](https://docs.npmjs.com/files/package.json) in more detail.

T> You can set those `npm init` defaults at *~/.npmrc*.

T> This is a good place to set up version control using [Git](https://git-scm.com/). You can create a commit per step and tag per chapter, so it’s easier to move back and forth if you want.

## Installing Webpack

Even though webpack can be installed globally (`npm install webpack -g`), I recommend maintaining it as a dependency of your project to avoid issues, as then you will have control over the exact version you are running.

The approach works nicely in **Continuous Integration** (CI) setups as well. A CI system can install your local dependencies, compile your project using them, and then push the result to a server.

To add webpack to our project, execute:

```bash
npm install webpack --save-dev # -D if you want to save typing
```

You should see webpack at your *package.json* `devDependencies` section after this. In addition to installing the package locally below the *node_modules* directory, npm also generates an entry for the executable.

## Executing Webpack

You can display the exact path of the executables using `npm bin`. Most likely it points at *./node_modules/.bin*. Try running webpack from there through the terminal using `node_modules/.bin/webpack` or a similar command.

After running, you should see a version, a link to the command line interface guide and an extensive list of options. We won’t be using most of those, but it’s good to know that this tool is packed with functionality if nothing else.

```bash
webpack-demo $ node_modules/.bin/webpack
No configuration file found and no output filename configured via CLI option.
A configuration file could be named 'webpack.config.js' in the current directory.
Use --help to display the CLI options.
```

To get a quick idea of webpack output, try this:

1. Set up *app/index.js* so that it contains `console.log('Hello world');`.
2. Execute `node_modules/.bin/webpack app/index.js build/index.js`.
3. Examine *build/index.js*. You should see webpack bootstrap code that begins executing the code. Below the bootstrap you should find something familiar.

T> We can use `--save` and `--save-dev` to separate application and development dependencies. The former will install and write to *package.json* `dependencies` field whereas the latter will write to `devDependencies` instead.

## Directory Structure

To move further, we can implement a little site that loads JavaScript, which we then build using webpack. After we progress a bit, we’ll end up with a directory structure like this:

- app/
  - index.js
  - component.js
- build/
- package.json
- webpack.config.js

The idea is that we’ll transform that *app/* to as a bundle below *build/*. To make this possible, we should set up the assets needed and *webpack.config.js*.

## Setting Up Assets

As you never get tired of `Hello world`, we might as well model a variant of that. Set up a component like this:

**app/component.js**

```javascript
export default function (text = 'Hello world') {
  const element = document.createElement('div');

  element.innerHTML = text;

  return element;
}
```

Next, we are going to need an entry point for our application. It will `require` our component and render it through the DOM:

**app/index.js**

```javascript
import component from './component';

document.body.appendChild(component());
```

## Setting Up Webpack Configuration

We’ll need to tell webpack how to deal with the assets we set up. For this purpose, we’ll develop a *webpack.config.js* file. Webpack and its development server will be able to discover this file through a convention.

To keep things convenient to maintain, we’ll be using [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin) to generate an *index.html* for our application. *html-webpack-plugin* wires up the generated assets with it. Install it to the project:

```bash
npm install html-webpack-plugin --save-dev
```

At a minimum, it is nice to have at least `entry` and `output` fields in your configuration. Often you see a lot more as you will specify how webpack deals with different file types and how it resolves them.

Entries tell webpack where to start parsing the application. In multi-page applications, you may have an entry per page. Or you could have a configuration per entry as discussed later in this chapter.

It is good to note that all output related paths that you see in the configuration are resolved against `output.path` meaning that if you had an output related option somewhere and wrote `styles/[name].css` to it, webpack would output the `styles` directory below the output path.

To illustrate how to connect `entry` and `output` with *html-webpack-plugin*, consider the code below:

**webpack.config.js**

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

const PATHS = {
  app: path.join(__dirname, 'app'),
  build: path.join(__dirname, 'build'),
};

module.exports = {
  // Entry accepts a path or an object of entries.
  // We'll be using the latter form given it's
  // convenient with more complex configurations.
  //
  // Entries have to resolve to files! It relies on Node
  // convention by default so if a directory contains *index.js*,
  // it will resolve to that.
  entry: {
    app: PATHS.app,
  },
  output: {
    path: PATHS.build,
    filename: '[name].js',
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'Webpack demo',
    }),
  ],
};
```

The `entry` path could be given as a relative one. The [context](https://webpack.js.org/configuration/entry-context/#context) field can be used to configure that lookup. Given plenty of places expect absolute paths, I prefer to use absolute paths everywhere to avoid confusion.

T> I use **trailing commas** in the book examples on purpose as it gives cleaner diffs for the code examples. I will show you how to enforce this rule in the *Linting JavaScript* chapter.

T> `[name]` is a placeholder. Placeholders are discussed in detail in the *Adding Hashes to Filenames* chapter.

T> I like to use `path.join`, but `path.resolve` would be a good alternative. See the [Node path API](https://nodejs.org/api/path.html) for further details.

If you execute `node_modules/.bin/webpack`, you should see output:

```bash
Hash: 3f76ae042ff0f2d98f35
Version: webpack 2.2.1
Time: 376ms
     Asset       Size  Chunks             Chunk Names
    app.js    3.13 kB       0  [emitted]  app
index.html  180 bytes          [emitted]
   [0] ./app/component.js 148 bytes {0} [built]
   [1] ./app/index.js 78 bytes {0} [built]
Child html-webpack-plugin for "index.html":
       [0] ./~/lodash/lodash.js 540 kB {0} [built]
       [1] (webpack)/buildin/global.js 509 bytes {0} [built]
       [2] (webpack)/buildin/module.js 517 bytes {0} [built]
       [3] ./~/html-webpack-plugin/lib/loader.js!./~/html-webpack-plugin/default_index.ejs 540 bytes {0} [built]
```

The output tells us a lot. I’ve annotated it below:

* `Hash: 3f76ae042ff0f2d98f35` - The hash of the build. You can use this to invalidate assets through `[hash]` placeholder. We’ll discuss hashing in detail in the *Adding Hashes to Filenames* chapter.
* `Version: webpack 2.2.1` - Webpack version.
* `Time: 377ms` - Time it took to execute the build.
* `app.js    3.13 kB       0  [emitted]  app` - Name of the generated asset, size, the IDs of the **chunks** into which it is related, status information telling how it was generated, the name of the chunk.
* `index.html  180 bytes          [emitted]` - Another generated asset that was emitted by the process.
* `[0] ./app/component.js 148 bytes {0} [built]` - The ID of the entry asset, name, size, entry chunk ID, the way it was generated.
* `Child html-webpack-plugin for "index.html":` - This is plugin-related output. In this case *html-webpack-plugin* is doing the output of its own.

Examine the output below `build/`. If you look closely, you can see the same IDs within the source. To see the application running, open the `build/index.html` file directly through a browser. On macOS `open ./build/index.html` works.

T> It can be convenient to use a tool like *serve* (`npm install serve -g`) to serve the build directory. In this case, execute `serve` at the output directory and head to `http://localhost:3000` at your browser. You can configure the port through the `--port` parameter.

T> If you want webpack to stop execution on the first error, set `bail: true` option. Setting it kills the entire webpack process. The behavior is desirable if you are building in a CI environment.

T> In addition to a configuration object, webpack accepts an array of configurations as discussed in the *Build Targets* chapter. You can also return a `Promise` and eventually `resolve` to a configuration. If you have to fetch configuration related values before returning it, this can be valuable.

## Adding a Build Shortcut

Given executing `node_modules/.bin/webpack` is a little verbose, we should do something about it. npm and *package.json* double as a task runner through configuration. Adjust it as follows:

**package.json**

```json
...
"scripts": {
  "build": "webpack"
},
...
```

Run `npm run build`. You should see the same output as before.

This works because npm adds *node_modules/.bin* temporarily to the path. As a result, rather than having to write `"build": "node_modules/.bin/webpack"`, we can do `"build": "webpack"`.

You can execute this kind of scripts through *npm run*. If you run it as is, it will give you the listing of available scripts.

T> There are shortcuts like *npm start* and *npm test*. We can run these directly without *npm run* although that will work too. For those in a hurry, you can use *npm t* to run your tests.

T> It is possible to execute *npm run* anywhere within the project. It doesn’t have to be run in the project root to work. npm will figure out the project root for you.

## *html-webpack-plugin* Extensions

[html-webpack-template](https://www.npmjs.com/package/html-webpack-template) or [html-webpack-template-pug](https://www.npmjs.com/package/html-webpack-template-pug) complement *html-webpack-plugin* and provide more powerful templates to use with it.

There are also specific plugins that extend *html-webpack-plugin*'s functionality. I’ve listed examples of these below:

* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin) is able to generate favicons.
* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin) gives you more control over script tags and allows you to tune script loading further.
* [html-webpack-cdn-plugin](https://www.npmjs.com/package/html-webpack-cdn-plugin) makes it easier to use popular open source CDNs with it.
* [multipage-webpack-plugin](https://www.npmjs.com/package/multipage-webpack-plugin) builds on top of *html-webpack-plugin* and makes it easier to manage multi-page configurations.
* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin) adds [resource hints](https://www.w3.org/TR/resource-hints/) to your HTML files to speed up loading time.
* [preload-webpack-plugin](https://www.npmjs.com/package/preload-webpack-plugin) enables `rel=preload` capabilities for scripts. This helps with lazy loading and it combines well with techniques discussed in the *Building* part of this book.

## Conclusion

Even though we’ve managed to get webpack up and running, it’s not that much yet. Developing against it would be painful. Each time we wanted to check out our application, we would have to build it manually using `npm run build` and then refresh the browser. That’s where webpack’s more advanced features come in.

To recap:

* It is a good idea to use a locally installed version of webpack over a globally installed one. This way you can be sure of what version you are using. The local dependency works also in a Continuous Integration environment.
* Webpack provides a command line interface. You can use it even without configuration, but then you are limited by the options it provides.
* To write more complicated setups, you will most likely have to write a separate *webpack.config.js* file.
* *html-webpack-plugin* can be used to generate an HTML entry point to your application. Later in the book, you will see how to generate multiple separate pages using. The *Multiple Pages* chapter covers that.
* It is handy to use npm *package.json* scripts to manage webpack. You can use it as a light task runner and use system features outside of webpack.

 We’ll look into enabling automatic browser refresh in the next chapter to make the development experience of our setup nicer.
