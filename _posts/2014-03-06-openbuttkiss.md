---
title: openbuttkiss
layout: post
category: programming
custom_sub: max/msp
custom_bla: ""
---
<div class="toptop"></div>

<span style="text-align:center;">openbuttkiss</span>

<div style="margin-left:12px;margin-right:12px;" markdown="1">

### <u>Installation</u>

You can download openbuttkiss at github. Don't forget about npm install!

There are two different ways to install a plugin, a procedure for each generator in Photoshop.

1. One way is to drop the openbuttkiss plugin into the Photoshop plugin folder, located inside the actual photoshop application directory:

    : **Adobe Photoshop CC/Plug-ins/Generator**

    Now for Photoshop to recognize this plugin, the built-in version of generator must be turned on. To do this, go to photoshop's preferences. Under the plug-ins tab, you will see an option to enable generator.You may need to restart for this to take effect.

    Openbuttkiss should now be available to select from:

    : **File --> Generate --> Openbuttkiss**

2. The other way is to run openbuttkiss from a downloaded version of the generator-core repository. https://github.com/adobe-photoshop/generator-core. My workflow consists of having all the plugins I work on and modify inside the generator-core/test/plugins. In terminal, I then cd to generator-core and run:

    : **node app.js -f test/plugins/openbuttkiss.**

Be aware that conflicts may arise when more than one generators are running. I had to turn of the built in generator to the openbuttkiss refered to in this step to work.

### <u>Running</u>

Now that we have installed openbuttkiss, we are ready to run it. We will need to find a program that can handle sending/recieving osc messages. The two I use most are Osculator and Max/MSP. Eventually, later posts will be using Max/MSP exclusively, as it will give us some exciting new ways to influence our photoshop scripts. Since this is the first tutorial, we will use Osculator. It is free and fully functional to download and try, minus the 20 second timeouts.

**http://www.osculator.net/**

After installing, open osculator. Click on the "parameters" icon or alternatively go to view --> parameters. This page is where we store any osc address/targets that we plan on communicating too. Double click on the one of the targets empty address parameter (under the column where it says "OSC URL or choice from gear menu"). Type in:

**osc.udp://localhost:3333**

Close the parameter page. Now we will create the osc message that we will send. Go to Routing --> Manually Create Message... Name it whatever you would like and hit ok; a new message appears in the application window. Osculator is not a program used only sending and interpretting for osc messages, but also midi, applescript, joysticks, wii. Therefor we need to specify that this message will be dealing with osc. Click and hold the empty slot under the "event type" label/column, then select "OSC Routing. To select the osc address we specified earlier, click and hold the empty slot under the "value" column and select new. This will bring up the Parameters page so that we can select our desired target (osc.udp://localhost:3333 - assigned to 1/default in the video).

That's it, we should be set to send osc messages to photoshop. Be sure that the openbuttkiss plugin is checked. If unchecked, the messages sent will be ignored :(

### <u>NodeJS Code</u>

My feelings on this tutorial. Aims of this tutorial. Not to explain everything but to go over the details of the osc implementation and to clairfy some things (node-osc, nodejs, generator-core) that I found confusing and that stumbled me.

#### <u>package.json</u>

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

The "generator-core-version" field tells us what version of generator we are using. In my experience, using a version below 2 would render the plugin incompatible.

: **Potential problem with plugin at '/Users/samenglander/Github/generator-core/test/plugins/myopenbuttkiss': The plugin openbuttkiss is incompatible with this version of generator-core. generator-core version: 2.0.3-dev, plugin compatibility: ~1**

I recommend sticking with "~2".

Any dependencies that our plugin requires are listed in the "dependencies" field. Openbuttkiss requires node-osc. This package.json feature is handy beacuse it allows for light-weight distribution of packages, where by the actual/physical code of the dependencies are ommitted from the packages/download. Installing these dependencies is done via npm (node package manager).

: **npm install**

This looks to our package.json and links us to any required dependencies. A variety of control is available to choose what versions of these dependencies are to be selected or restricted.

: **https://www.npmjs.org/doc/json.html#dependencies**

Similarly, if we were starting a new node.js project from scratch, having our dependencies listed within the package.json file would allow us to CD to our directory and perform an “npm install”. The node_modules folder would be created inside our respective node.js project. Without a package.json file specifying these dependencies, we would have to append our dependency name to the npm install command.

: **npm install node-osc**


#### <u>Main.js</u>

Much of my code was mixed and matched, scraping from the other photoshop generator tutorials. I used Tom Krcha's getting started plugin https://github.com/adobe-photoshop/generator-getting-started as a base for building the following osc features.

**var oscR = require('node-osc'),**

This require retrieves the logic involved in receiving OSC messages. It will then draw from the node-osc module, which exists in the node_modules folder in our openbuttkiss directory/plugin.

It is up to the module's creator, in this case, of node-osc, to properly format the package so that it can be used as a module. Assigning objects to module.exports enables access to those objects by using require(''). If we open up openbuttkiss/node_modules/node-osc/node_modules, we will see that node-osc is retrieving logic from other modules as well. One of these is osc-min, another osc package. This way of heiarchy dependencies, node_modules requiring node-modules requiring more node-modules, gets me thinking of russian-dolls or infinity mirrors. Maybe not the best analogies but think of these if they helps you understand.

: http://en.wikipedia.org/wiki/Matryoshka_doll

: http://en.wikipedia.org/wiki/Infinity_mirror

: http://thecreatorsproject.vice.com/creators/chul-hyun-ahn

In the node-osc readme it is stated that sending and receiving functionality is accessed from different places.

Sending OSC messages:
::
    var osc = require('node-osc');

Listening for OSC messages:
::
  var osc = require('./lib/osc');

This brought some confusion for me, because both sending and recieving logic exists within js file, osc.js. For a while I formatted my recieve require as require('./node_modules/node-osc/lib/osc.js'). Changing to require('node-osc') worked too.

For those who'd like to wrap their heads around this execution chain of requires, look below. Put this shit below in a separate tutorial.

: main.js . located in /openbuttkiss/ . var osc = require('node-osc');

    : node-osc is the name of the package we are directed to. We are then directed to index.js as it is listed by the "main" field in node-osc's package.json.

: index.js . located in /openbuttkiss/node_modules/node-osc/ . module.exports = require('./lib/osc.js')

    : In osc.js we see that two things are happening.

        1. requiring further logic from other dependencies/exports:
        : var min = require('osc-min');
        : var dgram = require('dgram');
        : var util = require('util');
        : var events = require('events');
        : var jspack = require('jspack').jspack;

        2. The wrapping of objects in exports, so that they can be accessed like in the step above - module.exports = require('./lib/osc.js'):
        : exports.Message = Message;
        : exports.Client = Client;
        : exports.Server = Server;

I will stop here. We don’t need to have all **this** in the tutorial. Find a way to make a separate, node js tutorial. Link to it from this one. You may have to build it. Building a nodejs module tutorial that you did!!


**oscServer = new oscR.Server(3333, '127.0.0.1'),**

This line initializes an osc server and assigns it to the variable oscServer. This occurs when the plugin in initilaized, via terminal, or if we are using the internal generator, when photoshop is started up. Don't get this confused with the selecting of plugin from the drop down menu, as the line exists in the scope of the centrally wrapped(?) function. This serve which we *will*, not yet, communicate with will be able to receive osc messages on localhost (127.0.0.1) port 3333.

**toggler = 0,**

This object gets assigned either a 0 or 1 based on openbuttkiss's menustate, whether is is checked or not. If 0, our osc server will ignore any incoming messages. If 1, our osc server will perform an action.

**psToFront = "app.bringToFront();";**

By sending this to evaluateJSX, the photoshop application is targeted and focused and brought to the front of any applications we also have open.

**var colorFlip = "var fColor = app.foregroundColor; app.foregroundColor = app.backgroundColor; app.backgroundColor = fColor;";**

This is a simple block of code swaps the foreground and background colors. It must be passed as a string through JSXfunction. As this is jsx code, we are not allowed to execute it as we normally would inside a .jsx file. It must be pased through _generator.evaluateJSXString(''), which we will see later. This stumbled me when I first started, you can see my discussion with Joel here:

: https://github.com/adobe-photoshop/generator-core/issues/138

~~~
**oscServer.on("message", function (msg, rinfo) {
    if(toggler==1 && msg[1]){
        actions(msg[1]);
    }
});**
~~~

oscServer.on is a listener for incoming osc messages. The plugin must be selected/checked once for the listener to initialize. After that it will **always be on**, choosing whether or not to perform actions based on toggler’s status. If 0, do nothing. If 1 do something. So... for the action to be called, the openbuttkiss must be checked and a message must be recieved. Two things! msg[1]==1???

There may be a better way to deal with this toggling listener and it's attached server.such as actually finding a way to turn on/off the oscServer's listener or oscServer itself based on whether the plugin is checked or not.

msg[1] corresponds to the value of our incoming messages the osc message. The message “bubbahjubbah” was sent from osculator to openbuttkiss, it could be broken down like this.

We are assumming that this is a toggled message.

msg:     msg[0]       msg[1]
on = [ 'bubbahjubbah', 1 ]
off =[ 'bubbahjubbah', 0 ]

I believe node-osc and other osc packages allow for more advanced parsing of these can osc messages, as osc messages can allow for pretty complicated features/handling. In this plugin all I care about is when the message value is equal to one. I'm not even interested in the message name. osc messages...I have never had the use for doing anything more complicated than that. When I want to do more complicated things, I just handle things in Max.

~~~
**function actions(x){
    _generator.evaluateJSXString(colorFlip);
    _generator.evaluateJSXString(psToFront);
}**
~~~

Called from our oscServer.on, this function executes two things, colorFlip followed by our psToFront. Remember, both of these must be passes to _generator.evaluateJSXString('').

~~~
**if(checked==true) {
    toggler = 1;
    console.log("OSC enabled!");
} else {
    toggler = 0;
    console.log("OSC disabled!");
}**
~~~

This logic exists inside the onMenuClicked() function. Each time openbuttkiss menu state changes (whether it's checked on or off) an if / else statement occurs setting the toggler variable in our init function to 0 or 1.




this will be on mac

http://makandracards.com/makandra/6683-markdown-kramdown-examples
