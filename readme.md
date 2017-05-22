# Oak
[![release](https://img.shields.io/badge/release-v2.1.0-green.svg)](https://github.com/OakLabsInc/oak/releases/tag/2.1.0)
[![node](https://img.shields.io/badge/node-v7.4.0-green.svg)](https://github.com/nodejs/node/releases/tag/v7.4.0)
[![electron](https://img.shields.io/badge/electron-v1.6.6-green.svg)](https://github.com/electron/electron/releases/tag/v1.6.6)
[![Coverage Status](https://coveralls.io/repos/github/OakLabsInc/oak/badge.svg?branch=master&t=zYcBU6)](https://coveralls.io/github/OakLabsInc/oak?branch=master)
[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-green.svg)](http://standardjs.com/)

A very opinionated kiosk UI application based on [electron](http://electron.atom.io).

# Rationale
Most of the `electron` project if focused around desktop application development, which is great! But when you are dealing with public computing (ATM machines, airline ticketing, movie theater ticket vendors, etc), you don't really need all the features a traditional desktop application requires.
The job of the `oak` module is to give a really easy way to make a kiosk application with modern web technology, so that it's repeatable, scalable, and easy to rapidly prototype for production.

# Install
```bash
npm i -g oak
```

# Quick Start
```
$ oak --help

  Usage: run [options] <url>

  Options:

    -h, --help                 output usage information
    -V, --version              output the version number
    -b, --background <String>  Hex background color for initial window. Example: #f0f0f0
    -D, --display <Number>     Display to use
    -f, --fullscreen           Full screen width and height
    -F, --frame                Show window frame
    -h, --height <Number>      Window height
    -i, --insecure             Allow insecure connections (not recommended)
    -k, --kiosk                Kiosk mode
    -n, --node                 Enable node integration
    -t, --ontop                Start window ontop of others
    -s, --show                 Show window on start
    -S, --shortcut [list]      Register shortcuts, comma separated. reload,quit
    -t, --title <String>       Window title
    -u, --useragent <String>   User-Agent string
    -v, --verbose              Set log level to info
    -w, --width <Number>       Window width
    -x, --x <Number>           Window X position
    -y, --y <Number>           Window Y position
```

You can use any URL you want to simply launch a fullscreen webpage, for example:
```
oak http://gifdanceparty.giphy.com/
```

# Making an app

`oak` only requires a couple things to get up and running: A URL, or an `index.js` file. You can specify a path to your module the same way you can with a URL:
```
oak path/to/app.js
```

## index.js
The most minimal example, this will launch a fullscreen app, injecting the `oak` object into the client side:
```
const oak = require('oak')

// when oak is ready, we can tell it to load something
oak.on('ready', () => {
  // loading takes an options object with a `url`, second parameter is an optional callback
  oak.load({
    url: 'http://www.mywebapp.com'
  }) // or callback)
})
```

## `require('oak')`
When you launch your app, the `oak` module is automatically resolved in modules, meaning you don't need to include it in your `package.json` file. This is similar to the way electron exposes it's own modules privately.

## `oak.load(options[, callback])`
Most of these options are wrapping electron.js `BrowserWindow` options, but some are specific to our kiosk use-case. This method returns the `Window` object
* `options`: Object
  * `url`: - Not optional
    * `String` - The `url` option is the only one required, and will load any valid URI
      ```
      // load a local HTML file
      url: 'file://' + join(__dirname, 'index.html')

      // load your own webserver
      url: 'http://localhost:8080'
      ```
    * `Function` - You can also pass a function to `url`. The first parameter is a callback which you pass your string to. This is helpful for dynamic page loading, redirecting to another page, or simply passing query parameters.
      ```
      url: function (callback) {
        callback('http://localhost:8080/?time=' + Date.now())
      }
      ```
  * `title`: String `OAK`- The window title
  * `display`: Number `0` - Your display number, and defaults to your main display
  * `fullscreen`: Boolean `true` - Set the window to max height and width
  * `kiosk`: Boolean `false` - Sets kiosk mode
  * `ontop`: Boolean `true` - Set the window to be always on top of others
  * `show`: Boolean `true` - Start the window shown
  * `height`: Number `1024`
  * `width`: Number `768`
  * `x`: Number `0` - X position
  * `y`: Number `0` - Y position
  * `shortcut` Object
    * `reload` Boolean `false` - enable CommandOrControl+Shift+R to reload the window
    * `quit` Boolean `false` - enable CommandOrControl+Shift+X to close the app
  * `background`: String `#000000` - Hex color of the window background
  * `frame`: Boolean `false` - Show window frame
  * `modules`: Array - Local node modules to load into the `window` during pre-dom phase
  * `flags`: Array - Chrome launch flags to set while starting the window
  * `insecure` Boolean - allow running and displaying insecure content (not recommended at all)
  * `waitForUrl`: Boolean `false` - On reload, waits for an explicit `proceed([url])` call, or `proceed` event
  * `userAgent`: String - Defaults to `'Oak/' + oak.version`
* `callback`: [Function] - Executed when the `ready` function has fired

## Methods
`oak.load()` returns a `Window` object with methods and events. The same methods are available in the client side on the `window.oak` object.

### `send(event[, payload])`
Send events to the window
* `event`: String - the event namespace, delimited by `.`
* `payload`: Any - whatever data you want to send along
Example: `window.send('myEvent', { foo: 'bar' })`

### `reload([url])`
Reload the window. If `waitForUrl` was passed, after this function is the time to use `proceed`
* `url`: String - Optional new URL to load, instead of the previous set URL. This will not overwrite options.

### `debug()`
Toggle the chrome debugger

### `show()`
Show the window

### `hide()`
Hide the window

### `focus()`
Set the desktop focus to this window

### `proceed([url])`
If `waitForUrl` was set `true`, you will need to execute this in order to continue with a reload.
* `url`: String - Optional new URL to load, instead of the previous set URL. This will not overwrite options.

### `on(event, callback)`
This is an instance of `EventEmitter2`
* `ready` - Will emit the ready event, and also execute the optional callback
* `reloading` - The window is reloading, can be used in tandem with `proceed()`
  * `oldSession` - previous session ID
  * `newSession` - new session ID
* `loadFailed` - The window load failed
  * `opts`: Object - original options used
  * `err`: Error

# Examples
* [`simple kiosk`](https://github.com/OakLabsInc/oak-examples/tree/master/simple-kiosk)
* [`tetris`](https://github.com/OakLabsInc/oak-examples/tree/master/tetris)
* [`multiple windows`](https://github.com/OakLabsInc/oak-examples/tree/master/multiple-windows)

# I don't always test my code, but when I do...

We use docker containers for all our production deployments. To get started, you will need to have Docker installed. You can install docker from their [website](https://www.docker.com/products/docker), or on Linux systems, run this script:

```sh
  curl -sSL https://get.docker.com/ | sh
  # add your user to the docker group
  sudo usermod -aG docker $(whoami)
```

You will also need an X server running (`xorg`). If you are on OSX, go ahead and follow the steps below to get setup.

## On Linux
This example is for debian based systems, you can accomplish the same by getting `docker` and `docker-compose` running yourself.

1. Install `python-setuptools`
   ```
    sudo apt-get install -y python-setuptools
   ```

2. Install `pip`
   ```
    sudo easy_install pip
   ```
3. Install `docker-compose`
   ```
    pip install docker-compose>=1.8.0
   ```

4. Allow your `X` server to allow outside connections. Make sure to disable this after you are finished!
   ```
    xhost +
    docker-compose up

   ```
    You should turn off your open xhost after you are finished developing.
   ```sh
    docker-compose down
    xhost -
   ```

## On OSX
I'm not going to lie... this is a pain in the ass. OSX doesn't have `xorg` or a build in X server by default. You are going to be using `socat` to proxy Xquartz via TCP so that you can use your IP address the docker container. It may be easier to start up a VM running ubuntu or debian.

1.    Install [homebrew](https://brew.sh/)

      Homebrew is a easy way to install linux packages on OSX. In your `Terminal` app:
      ```
      /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
      ```

2.    Install `socat` and `python`

      * `socat` will be needed to forward your X server socket, in order to display a window on your desktop.
         ```
         brew install socat
         ```
      * `python` is useful for a number of reasons, but in our case, a means to get `docker-compose`. When you install `python`, you get the `pip` program along with it.
        ```
         brew install python
        ```

3.    Install `docker-compose`

      Rather than using straight `docker` commands, we use `docker-compose` to simplfy orchestrating multiple containers. `docker-compose` uses a `.yml` file to describe docker commands and run them.
      ```
      pip install docker-compose
      ```

4.    Install [XQuartz](https://www.xquartz.org/), which is a X server for OSX.
      ​

5.    Open XQuartz, go to Preferences > Security > Allow connections from network clients.
      ​

6.    In `Terminal`, run `socat` to proxy your X server connection via TCP:
      ```
       ⁠⁠⁠⁠socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
      ```

       After you run this, it will be waiting for connections, so don't close this `Terminal` window.
      ​

7.    Edit `docker-compose.osx.yml`

       Replace the X's with your IP address. This will resolve your `socat` connection to the container, which is proxying XQuartz.
      ```
      environment:
        - DISPLAY=XXX.XXX.XXX.XXX:0
      ```

      If your IP address was `192.168.0.5`, the line would be `DISPLAY=192.168.0.5:0` . Don't forget the `:0` and the end, that specifys that it's the first display, not a port.

8. In your `oak` directory, run `docker-compose -f docker-compse.osx.yml up`

## On Windows
Sorry but you are a little on your own as far as an X server goes! In the future we may update this readme to provide info for developing on Windows. In the mean time... [Cygwin](https://www.cygwin.com/)?
