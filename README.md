# Testee

[![Greenkeeper badge](https://badges.greenkeeper.io/bitovi/testee.svg)](https://greenkeeper.io/)
[![NPM Version](https://img.shields.io/npm/v/testee.svg)](https://npmjs.org/package/testee)
[![Build Status](https://img.shields.io/travis/bitovi/testee.svg)](https://travis-ci.org/bitovi/testee)
[![Gitter Chat](https://img.shields.io/gitter/room/bitovi/testee.svg)](https://gitter.im/bitovi/testee)

> Automated cross-browser testing made easy.

- Supports any local browser that supports [socket.io](https://npmjs.com/socket.io)
- Supports BrowserStack, Electron and PhantomJS
- Supports Mocha, Jasmine and QUnit
- Code coverage
- Local and remote file testing

## Installation

[Node.js](http://nodejs.org/) `>= 4` is required. To install, type this at the command line:
```shell
npm install testee
```

You can choose to install globally with `-g` if you wish, but it is recommended that you install per-project so that CI environments and all members of your team have instant access to the same dependencies.


## Command Line Usage

On the command line, you have the following options available:

* `-h`, `--help`: output usage information
* `-V`, `--version`: output the version number
* `-b`, `--browsers` `[name]`: A comma separated list of browsers you want to run (default: `phantom`)
* `-R`, `--root [path|URL]`: The server root path or URL the files are relative to
* `-p`, `--port` `[port]`: The port to run the server on (default: `3621`)
* `-r`, `--reporter` `[name]`: The name of the reporter to use (default: `Dot`)
* `-c`, `--config` `[file]`: Use this JSON or JS configuration file (can be overridden by command line options)
* `--timeout` `[seconds]`: The per test timeout (in seconds)
* `--delay` `[ms]`: When running multiple tests, the time to wait for the browser to shut down before starting it with a new page.
* `-s`, `--server`: Only run the server
* `--coverage`: Enable code coverage

### Examples

Test with one or multiple files:
```shell
testee test.html
```
```shell
testee test1.html test2.html
```

Save keystrokes with a base path/URL:
```shell
testee test1.html test2.html --root=/var/www/app/
```
```shell
testee test1.html test2.html --root=http://yourapp/
```

Specify browsers:
```shell
testee test.html --browsers=firefox,safari
```

Use a [configuration file](#configuration-api):
```shell
testee test.html --config=testee.json
```
```shell
testee test.html --config=testee.js
```


## Development Flow

During development it is nice to have tests run when files change. Testee is unique in that you do not need Testee while developing. Test pages can be served locally by something like [`http-server`](https://github.com/indexzero/http-server) or [`live-server`](https://github.com/tapio/live-server) and opened/reloaded in the browser at will. This is the recommended way to iterate when using Testee. Testee is better used to test against multiple browsers as part of your build and/or release process, or alongside [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) for commits or pushes.


## Programmatic Usage

For custom scripts (including [gulp](https://npmjs.com/gulp)):

```js
const testee = require('testee');

const browsers = 'phantom';
const config = { reporter: 'spec' };
const files = ['test/test.html'];

testee.test(files, browsers, config)
  .then(function() {
    console.log('done!')
  });
```


## Grunt Usage

See the [Testee Grunt plugin `grunt-testee`](https://github.com/bitovi/grunt-testee) for more information.

## Configuration API

A simple, local browser config (JSON) example with mostly default values could look like:
```json
{
  "port": 3621,
  "root": "/var/www/app/",
  "reporter": "dot",
  "timeout": 120,
  "delay": 1000,
  "tunnel": {
    "type": "local"
  },
  "launch": {
    "type": "local"
  },
  "browsers": [
    "firefox",
    {
      "browser": "chrome",
      "args": [
        "--headless",
        "--disable-gpu",
        "--remote-debugging-port=9222"
      ]
    }
  ]
}
```

BrowserStack hosts virtual machines running specific versions of web browsers and is extremely useful for cross-browser testing. It will require a username and password. An advanced config (JS) that runs your tests on an iPad Mini and Samsung Galaxy S3 emulator using BrowserStack in a CI environment (outputting XUnit logs) could look like:
```js
require('dotenv').config();

const pkg = require('./package.json');

module.exports = {
    reporter: 'XUnit',
    coverage: {
      dir: 'coverage/',
      reporters: ['text', 'html'],
      ignore: ['node_modules']
    }
    tunnel: {
      type: 'browserstack',
      key: process.env.BROWSERSTACK_KEY
    },
    launch: {
      type: 'browserstack',
      username: process.env.BROWSERSTACK_USER,
      password: process.env.BROWSERSTACK_KEY,
      version: 4
    },
    browsers: [
      { project:pkg.name, build:pkg.version, os:'ios', device:'iPad Mini', os_version:'6.0' },
      { project:pkg.name, build:pkg.version, os:'android', device:'Samsung Galaxy S III', os_version:'4.1' }
    ]
  }
};
```

### General Settings

#### `delay`
Type: `Number`  
Default value: `1000`  
The delay (in milliseconds) between multiple test pages within a single browser.

#### `port`
Type: `Number`  
Default value: `3621`  
The port of the static fileserver used to serve the tests. This will also be used by [Localhost tunneling services](#localhost-tunneling).

#### `reporter`
Type: `String`  
Default value: `'dot'`  
See [Mocha reporters](https://mochajs.org/#reporters).

#### `root`
Type: `String`
Default value: `process.cwd()`  
The root path (or base URL) of the static fileserver used to serve the tests. Any test file will be relative to this path.

#### `timeout`
Type: `Number`  
Default value: `120`  
The time (in seconds) to wait for a test page to report back before an error is thrown. This timeout might, for example, occurr when the given file doesn't exist, the browser didn't start, or the localhost tunnel wasn't running.


### Browser Settings

Choose between locally installed browsers and those provided remotely by BrowserStack.

**Note:** Depending on your OS, the target browsers should not already be open/running before using this library. Electron and PhantomJS are the exceptions, as they can always be started multiple times.

#### `browsers`
Type: `Array`  
Default value: `['phantom']`  
The browsers that will be used to run tests. For local browsers, use a browser name string (see [Launchpad](https://npmjs.com/launchpad) for more info). For a remote/BrowserStack browser, use a [browser object](https://npmjs.com/browserstack#browser-objects). To pass command-line arguments and other browser options to Launchpad, use an object containing the browser name and optional array of arguments and options. This browser object will spawn a Chrome Headless browser:

```json
{
  "browser": "chrome",
  "args": [
    "--headless",
    "--disable-gpu",
    "--remote-debugging-port=9222"
  ]
}
```

#### `launch`
Type: `Object`  

#### `launch.password`
Type: `String`  
Default value: `undefined`  
Your BrowserStack API key.

#### `launch.type`
Type: `String`  
Default value: `'local'`  
The test environment. Possible values are `'local'` and `'browserstack'`.

#### `launch.username`
Type: `String`  
Default value: `undefined`  
Your BrowserStack username.

#### `launch.version`
Type: `String`  
Default value: `undefined`  
The BrowserStack API version you'd like to use. The recommended value is `4`.


### Code Coverage

These options are used to instrument and report code coverage using [Istanbul](https://github.com/gotwarlost/istanbul).

#### `coverage`
Type: `Object`  

#### `coverage.dir`
Type: `String`  
Default value: `'./coverage'`  
The directory where the coverage data should be written. `text` reports will be written to the console.

#### `coverage.ignore`
Type: `Array`  
Default value: `['text']`  
A list of regex patterns that match files to be ignored by coverage instrumentation and reporting.

**Note:** When using the `--coverage` option, `coverage.ignore` will default to `['node_modules']`.

#### `coverage.reporters`
Type: `Array`  
Default value: `[]`  
The type of reporter(s) to use. [Available reporters](https://github.com/gotwarlost/istanbul/tree/master/lib/report).

**Note:** [`babel-plugin-istanbul`](https://npmjs.com/babel-plugin-istanbul) instruments ES2015/ES6 code automatically.


### Localhost Tunneling

A localhost tunneling service makes your local system available to the outside world. This is great if you want to run tests on another system which can't easily reach your local machine. Such a service is necessary for giving BrowserStack workers an endpoint to communicate with.

#### `tunnel`
Type: `Object`  
Default value: `{ type: 'local' }`  
See [miner documentation](http://daffl.github.com/miner/) for all available tunneling services and options.


## Electron & PhantomJS

If you plan to use these, be sure to add [electron](https://npmjs.com/electron) and/or [phantomjs-prebuilt](https://npmjs.com/phantomjs-prebuilt) to your list of `devDependencies`, or have them installed globally on your system.

## Chrome Headless

To easily leverage Chrome Headless, you can add [`puppeteer`](https://github.com/GoogleChrome/puppeteer) to your project as a development dependency and the following browser option to your config's `browsers` array:

```js
const puppeteer = require('puppeteer')
process.env.LAUNCHPAD_CHROMIUM = puppeteer.executablePath()

module.exports = {
  "browser": "chromium",
  "args": [
    "--headless",
    "--disable-gpu",
    "--remote-debugging-port=9222"
  ]
}
```

## CI integration

Because different reporters are supported for the test result output, it is easy to obtain XUnit style XML files that integrate with CI servers like [Jenkins](http://jenkins-ci.org/). Just use the `'XUnit'` reporter and write the output into a file. The following example writes the XML result of a Firefox test to "testresults.xml":

```shell
testee test.html --browsers=firefox --reporter=XUnit > testresults.xml
```

See [available reporters](#reporter).

**Note:** If your CI platform uses Ubuntu to run builds, such as Travis CI, additional configuration is needed for FireFox to run properly. See bellow for more details.

## Launching FireFox on Linux

The Linux version of FireFox uses [D-Bus](https://www.freedesktop.org/wiki/Software/dbus/) which is not preinstalled on all Linux distributions, including Ubuntu, this can cause errors when launching tests, heavily pollute Testee's debug logs with error messages, and make FireFox run slow.

To solve this problem, we recommend installing `dbus-x11` via the `apt` package manager before the build executes.

Here is an example configuration for Travis CI:
```yml
language: node_js
node_js:
  - "6"
addons:
  firefox: "latest-esr"
  # Install D-Bus here
  apt:
    packages:
      - "dbus-x11"
before_install:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start"
```

## Capturing `console.log` and `console.error`

All calls to `console.log` and `console.error` in a test are tracked. To get the output during a test run, set the `DEBUG` environment variable to `testee:console-log`:

```shell
DEBUG=testee:console-log testee test.html --browsers=canary
```

## Debugging

Detailed debugging information can be enabled in any environment (command line, Grunt, programatically) by setting the `DEBUG` environment variable to `testee:*`:

```shell
DEBUG=testee:* testee test.html --browsers=canary
```


## Client side configuration

In most cases there is no need to change your actual test code.

One exception is when you load your testing library using an asynchronous client side loader like Steal or RequireJS because Testee won't know which library adapters to attach. In this case, you need to call `Testee.init()` manually once the test library is loaded:

```html
<script>
  define(['qunit'], function() {
    // Needs to check because it will only be available
    // when running the test with Testee
    if(window.Testee) {
      window.Testee.init();
    }

    QUnit.start();
  });
</script>
```

In some testing environments, reporting test progress via REST may work better than socket.io:

```html
<script>
  window.Testee = {
    provider: {
      type: 'rest'
    }
  };
</script>
```

## Troubleshooting

#### Browser caching
On Safari, disable caching by choosing `Develop -> Disable Caches` and also `Develop -> Empty Caches`. Without these preferences, updated tests may not load in the browser.

#### "UnhandledPromiseRejectionWarning" messages
If you see a message which looks like the one below, there is most likely an incorrect assertion somewhere in your tests. Please refer to [issue #136](https://github.com/bitovi/testee/issues/136) for an example. If the warning is unrelated to your assertions, please open an issue.

```
UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1):
Error: Test `title` should be a "string" but "number" was given instead.
```
