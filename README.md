Grunt support
=============
This buildpack uses the official Node.js buildpack refactoring called "diet" (faster compilation),
and incorporate installation of compass and auto-execution of grunt.

The command that runs is `grunt heroku`.

Heroku Buildpack for Node.js
============================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for Node.js apps. It will detect your app as Node.js if it has a `package.json` file in the root. It uses npm to install your dependencies, and vendors a version of the Node.js runtime into your slug.

If you specify a version of node in the [`engines` field of your package.json](https://npmjs.org/doc/json.html#engines), the buildpack will attempt to find the specified version on [nodejs.org/dist](http://nodejs.org/dist/) and download it from our S3 caching proxy.

If you don't specify a version of node, the latest stable version will be used.

About this Refactor
-------------------

This branch of the buildpack is intended to replace the [official Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs#readme) once it has been tested by some users. To use this buildpack for your node app, simply change your BUILDPACK_URL [config var](https://devcenter.heroku.com/articles/config-vars) and push your app to heroku.

```
heroku config:set BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-nodejs#diet -a my-node-app
git commit -am "fakeout" --allow-empty
git push heroku
```

Here's a summary of the differences between the current official buildpack and this _diet fork_ version:

The old buildpack:

- Contains a lot of code for compiling node and npm binaries and moving them to S3. This code is orthogonal to the core function of the buildpack, and is only used internally by Node maintainers at Heroku.
- Downloads and compiles node and npm separately.
- Requires manual intervention each time a new version of node or npm is released.
- Does not support pre-release versions of node.
- Uses SCONS to support really old versions of node and npm.
- Maintains S3 manifests of our hand-compiled versions of node and npm.
- Does not cache anything.

The new buildpack:

- Uses the latest stable version of node and npm by default.
- Allows any recent version of node to be used, including pre-release versions, as soon as they become available on [nodejs.org/dist](http://nodejs.org/dist/).
- Uses the version of npm that comes bundled with node instead of downloading and compiling them separately. npm has been bundled with node since [v0.6.3 (Nov 2011)](http://blog.nodejs.org/2011/11/25/node-v0-6-3/). This effectively means that node versions `<0.6.3` are no longer supported, and that the `engines.npm` field in package.json is now ignored.
- Makes use of an s3 caching proxy of nodejs.org for faster downloads of the node binaries.
- Makes fewer HTTP requests when resolving node versions.
- Uses an updated version of [node-semver](https://github.com/isaacs/node-semver) for dependency resolution.
- No longer depends on SCONS.
- Caches the `node_modules` directory across builds.
- Runs `npm prune` after restoring cached modules, to ensure that any modules formerly used by your app aren't needlessly installed and/or compiled.

This fork:

- Allows you to configure the location of the application inside the project
- Allows you to specify the npm command to run.

Documentation
-------------

For more information about buildpacks and Node.js, see these Dev Center articles:

- [Heroku Node.js Support](https://devcenter.heroku.com/articles/nodejs-support)
- [Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/nodejs)
- [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [Buildpack API](https://devcenter.heroku.com/articles/buildpack-api)

Specific to this fork
---------------------

Grunt, grunt-cli, and bower are installed for you so they do not have to be included in project dependencies. `bower install` is run to install bower dependencies.

Specific to the treasure-data/heroku-buildpack-nodejs-grunt-compass-configurable fork
-------------------------------------------------------------------------------------
This fork can take an additional config file which allows you to specify where to find the `package.json`. The gruntfile (`grunt.js`, `Gruntfile.js`, `Gruntfile.coffee`) must be in the same directory. The config file should be `.heroku_config` and located in the root of your project.

Example `.heroku_config` file:

    export NODE_WORKING_DIRECTORY='/console'
    export NPM_COMMAND='npm install'

- `NODE_WORKING_DIRECTORY`- The location from the root of your project where you can find the `package.json` and gruntfile (`grunt.js`, `Gruntfile.js`, `Gruntfile.coffee`). If it is not specified it will look in the root of your application. The string value is just appended to the value that is passed to the bin/compile and bin/detect scripts, so be sure to prefix it with a forward slash.
- `NPM_COMMAND` - The command that should be run to setup your dependencies. It defaults to `npm install --production` if you do not specify anything. We `eval` this string, so procede with caution.

These commands are all executed from the directory you specify with `NODE_WORKING_DIRECTORY`, and you can access the current location using `$build_dir`.

Hacking
-------

To make changes to this buildpack, fork it on Github. Push up changes to your fork, then create a new Heroku app to test it, or configure an existing app to use your buildpack:

```sh
# Create a new Heroku app that uses your buildpack
heroku create --buildpack <your-github-url>

# Configure an existing Heroku app to use your buildpack
heroku config:set BUILDPACK_URL=<your-github-url>

# You can also use a git branch!
heroku config:set BUILDPACK_URL=<your-github-url>#your-branch
```

For more detailed information about testing buildpacks, see [CONTRIBUTING.md](CONTRIBUTING.md)
