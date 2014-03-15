---
title: openbuttkiss
layout: post
category: programming
custom_sub: max/msp
---
<!--# <span class="cspan" markdown="">[ openbuttkiss ] OSC and Photoshop Communication</span>-->

<!--## Introduction to Openbuttkiss-->

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

    (picture)

2. Now let's create the osc message we plan to send. Go to `Routing > Manually Create Message...` Name it whatever you want and hit ok. A new message appears in the application window.

3. Under the Event Type column, select `OSC Routing` from the dropdown menu (click/hold the empty slot of the message to reveal the dropdown menu).

4. Under the Value column, select `Đ` (default) from the dropdown menu. Our osc message should now be targeting the osc address specified earlier.

Done. We should now be able to send osc messages to Photoshop. Remember that the openbuttkiss plugin must be checked in order to respond to incoming osc messages and trigger the swap foreground and background color logic (put in overview, what the patch does on osc incoming messages). If unchecked, osc messages will be ignored?

## Plugin Code

### package.json

~~~
{
    "name": "openbuttkiss",
    "description": "osc-ps plugin",
    "version": "0.0.0",
    "main": "main.js",
    "generator-core-version": "~2",
    "dependencies": {
        "node-osc": "0.2.1"
    },
    "devDependencies": {}
}
~~~

Determined by the "main" field in package.json, main.js will be the primary entry point of openbuttkiss.

The "generator-core-version" field tells us what version of generator we are using. In my experience, a version below 2 would render the plugin incompatible.

{:.break-me}
~~~
Potential problem with plugin at '/Users/samenglander/Github/generator-core/test/plugins/myopenbuttkiss': The plugin openbuttkiss is incompatible with this version of generator-core. generator-core version: 2.0.3-dev, plugin compatibility: ~1
~~~

I recommend sticking with "~2".

Any dependencies that our plugin requires are listed in the "dependencies" field. There were all downloaded into our plugin folder when we ran `npm install` Openbuttkiss requires node-osc. Node-osc including osc-min facts.

### main.js

Much of my code was scraped/taken from Tom Krcha's [getting-started-plugin](https://github.com/adobe-photoshop/generator-getting-started "getting-started-plugin"). It provided me a foundation for building up (adding) the/these following osc features.

    var oscR = require('node-osc'),

* This require retrieves the logic involved in receiving OSC messages. It will then draw from the node-osc module, which exists in the node_modules folder in our openbuttkiss directory/plugin. ANYTHING ELSE?
^
    oscServer = new oscR.Server(3333, '127.0.0.1'),

* Creates an osc server and assigns it to the variable oscServer. This server initialization will happen when we run the plugin from terminal, or, if using the internal generator, whenever we start up Photoshop. Once we set up the listerner, our server will be able to recieve incoming osc messages at localhost (127.0.0.1) port 3333. <u>Don't get this mixed up with our server listener, they are two different things.</u>
^
    toggler = 0,

* This object gets assigned either a 0 or 1 based on openbuttkiss's menu-state - whether is is checked or unchecked. 0 = unchecked, 1 = checked.
^
    psToFront = "app.bringToFront();";

* By sending this to evaluateJSX, the photoshop application is targeted and brought to the front of any applications we also have open.
^

{:.break-me}
    var colorFlip = "var fColor = app.foregroundColor; app.foregroundColor = app.backgroundColor; app.backgroundColor = fColor;";

* This simple block of code swaps the foreground and background colors. As this is jsx code, we must treat it differently than the other javascript code in main.js. It must be passed as a string through _generator.evaluateJSXString(''). I got majorly stuck on this when I first tried to do anything with JSX code and Generator - [this is me getting help](https://github.com/adobe-photoshop/generator-core/issues/138 "thx joel").
^

~~~
oscServer.on("message", function (msg, rinfo) {
    if(toggler==1 && msg[1]==1){
        actions();
    }
});
~~~

* oscServer.on is the server's listener - it listens for incoming osc messages. <u>The plugin must be checked atleast once for the listener to initialize</u>. After that it will always be on, choosing whether or not to perform actions based on toggler’s status. If 0, the message is recieved, but nothing is done. If 1, our actions() function is called. So in this plugin, the conditions that must be met for our actions to execture are: openbuttkiss must be checked and an osc message with the value 1 must be recieved.

    There are probebly other ways in which (determine) our plugin menu-state can influence our osc server/listener. Finding a way to actually disable the oscServer's listener or oscServer itself based on menu-state may be a more efficient solution to what we have currently: the server listener always being on, recieving osc messages all the time, only executing an action when the menu-state is checked.

#### Understanding OSC Messages

Node-osc gives us access to different parts of our osc message. Msg[0] retrieves the address, while msg[1] retrieves the argument. The message “bubbahjubbah”, sent from osculator to openbuttkiss, could be broken down like this:

Note that this is a toggle message, transmitting two arguments each time it is sent. 1 = on, 0 = off.

|Osc Message|msg[0]|msg[1]|
|:-:|:-:|:-:|
|[ 'bubbahjubbah', 1 ]|bubbahjubbah|1|
|[ 'bubbahjubbah', 0 ]|bubbahjubbah|0|

I'm sure node-osc and other nodejs osc modules allow for more elaborate parsing of messages. I have yet to delve deeper into opensoundcontrol, and my involvment with it has probebly not surpassed a most basic usage. The benefits of osc seem to be in its ability to develop a rich organization of flexible parameter names. Openbuttkiss keeps clear of all that - its only interest is a message argument of 1; It places no importance on the message address, which can be called anything.

For more info on opensoundcontrol

: [Open Sound Control 1.0 Specification](http://opensoundcontrol.org/spec-1_0 )

: [Features and Future of Open Sound Control 1.1](http://cnmat.berkeley.edu/system/files/attachments/Nime09OSCfinal.pdf )

: [Control of VST Plug-ins Using OSC](http://opensoundcontrol.org/files/zbyszynski_ICMC3.pdf )

~~~
function actions(){
    _generator.evaluateJSXString(colorFlip);
    _generator.evaluateJSXString(psToFront);
}
~~~

Called from our oscServer.on, this function executes two things, colorFlip followed by our psToFront. Remember, both of these must be passed to _generator.evaluateJSXString().

~~~
if(checked==true) {
    toggler = 1;
    console.log("OSC enabled!");
} else {
    toggler = 0;
    console.log("OSC disabled!");
}
~~~

This logic exists inside the onMenuClicked() function. Each time openbuttkiss's menu-state changes (checked/unchecked), an if / else statement is executed, setting toggler to 0 or 1.

this will be on mac

http://makandracards.com/makandra/6683-markdown-kramdown-examples

My feelings on this tutorial. Aims of this tutorial. Not to explain everything but to go over the details of the osc implementation and to clairfy some things (node-osc, nodejs, generator-core) that I found confusing and that stumbled me.

### Requirements

other node osc modules
https://nodejsmodules.org/tags/osc


the or our
the/our
THE/OUR
????

Clicking the green box in osculator!!!

USE ARGUMENT>> NOT VALUE !!
