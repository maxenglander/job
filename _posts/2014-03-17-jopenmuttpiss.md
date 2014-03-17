---
title: intro to max and ps
layout: post
category: programming
custom_sub: max/msp
custom_bla:
---
# Topics

1. Installing Node.js on Mac OS X with Brew
1. How does the Node.js search path work?
1. How do I decide whether to install something locally or globaly?
1. Look at OSC code
1. Look at Andy Hall's Photoshop code
1. Bare minimum Photoshop plugin

# Installing Node.js on Mac OS X with Brew

[Screencast](http://screencast.com/t/FXe6EvF0q8)

 1. Open terminal
 2. Type `brew install node`

# How does the Node.js search path work

[Loading modules from node_modules folders](http://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders)
[Loading modules from global folders](http://nodejs.org/api/modules.html#modules_loading_from_the_global_folders)

When you invoke the `require()` command in a Node.js program, e.g.

    var bk = require('buttkiss')

Node.js will search for the `buttkiss` module using the following algorithm

 1. If the `buttkiss` is a native module, Node.js will just load it - you probably don't need to know any more than that
 1. Node.js will search for the `buttkiss` module in `./node_modules/`
 1. Node.js will search for the `buttkiss` module in `../node_modules/`
 1. Node.js will continue searching for `buttkiss` in the `node_modules` folder by traversing upwards in the filesystem hierarchy
 1. If the `$NODE_PATH` environment variable is set, then Node.js will search the paths delimited in that environment variable
 1. Node.js will also search in `$HOME/.node_modules`, `$HOME/.node_libraries`, `$PREFIX/lib/node`

# How do I decide whether to install something globally or locally?

 1. Any module that a project depends on should be expressed as a dependency in `package.json`, and will therefore be installed locally when you run `npm install`
 1. Any module that is not a project dependency can be installed globally, e.g. a tool that watches your project for any filesystem changes and automatically runs your unit tests

# Look at OSC code

[Screencast](http://screencast.com/t/tCjhUUYFQ)

# Bare minimum Photoshop plugin

 1. Enable remote connections in Photoshop, using password "password"
 1. Open terminal
 1. `cd ~`
 1. Download generator-core with `git clone git@github.com:adobe-photoshop/generator-core`
 1. Download buttkiss with `git clone git@github.com:maxenglander/buttkiss`
 1. Install buttkiss with `cd generator-core; node app ../buttkiss`

## Installation

1. Download the [openbuttkiss]() plugin.

2. In terminal, cd to the plugin folder and type `npm install`. This installs any required dependencies

3. Follow one of the methods below to set up openbuttkiss with Photoshop Generator.

-Built-in Generator

: To use the built-in Generator, drop the openbuttkiss plugin into the Photoshop plug-ins folder. This is located inside the Photoshop application directory:

        Applications/Adobe Photoshop CC/Plug-ins/Generator

    For Photoshop to recognize this plugin, the built-in version of generator must be enabled. Go to Photoshop preferences, and under the plug-ins tab, `enable generator`. You may need to restart Photoshop for this to take effect.

-External Generator

: To use the external Generator, run the plugin from a local copy of the [generator-core](https://github.com/adobe-photoshop/generator-core) library. My workflow consists of having all my plugins inside the generator-core/test/plugins folder. In terminal, I then cd to generator-core and type:

        node app.js -f test/plugins/openbuttkiss

    This app.js contains the magic needed to run our plugin. The plugin is specified at the end of the command.

Photoshop has multiple generators that can be used to process a plugin, each providing a slightly different workflow. The plugin, regardless of which generator is being used, should yield the same results. Be aware that conflicts may arise when using both internal and external generators. In my experience, I had to disable the built-in generator to properly run openbuttkiss with the external generator.

[Read more on Photoshop Generators and the Generator Architecture](https://github.com/adobe-photoshop/generator-core/wiki/Generator-Architecture).

## Running

After completing the above steps, openbuttkiss should now be available. In Photoshop, select the plugin:

    File > Generate > Openbuttkiss

(two pictures: internal / external lists of plugins.. maybe combined into one picture?)

Now that we have installed openbuttkiss, we are ready to run it. We will need to find a program that can handle sending/recieving osc messages. The two I use are Osculator and Max/MSP. Later posts use will use Max/MSP as it gives us exciting ways to influence our photoshop workflow. Here, I use Osculator, which is free and fully functional to download, minus the 20 second timeouts.

### [Osculator Website](http://www.osculator.net/ "Osculator")

### Configuring Osculator

1. Open up Osculator. Click on the `Parameters` icon or go to `View > Parameters`. This page stores any osc addresses we plan on communicating to, with each address, being stored in a target. Under the <b>OSC URL or choice from gear menu</b> column, double click on the empty address parameter of target #1. Type in:

        osc.udp://localhost:3333

    Then, click the `+` symbol located in the bottom left corner of the Parameters page. Close the parameter page.
