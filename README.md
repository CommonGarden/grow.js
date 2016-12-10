# Grow.js

[![Join the chat at https://gitter.im/CommonGarden/Grow.js](https://badges.gitter.im/CommonGarden/Grow.js.svg)](https://gitter.im/CommonGarden/Grow.js?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Grow.js is an npm package for creating and connecting devices to a [Grow-IoT](https://github.com/CommonGarden/Grow-IoT) instance.

### Installation

```bash
npm install Grow.js
```

### Usage

Create a new device and take note of the device `uuid` and token.

In the `tests` folder checkout `test-device.js`.

Replace the `uuid` and `token` properties of the config object with the credentials you generate.

```javascript
// Import the latest build of the Grow.js library
var Thing = require('Grow.js');

// Create a new grow instance. Connects by default to localhost:3000
var testDevice = new Thing({
    // PUT YOUR UUID AND TOKEN HERE:
    uuid: 'PASTE_UUID_HERE',
    token: 'PASTE_TOKEN_HERE',
    
    // HACK, unfortunately needed for now...
    testDevice: true,

    // Properties can be updated by the API
    properties: {
        state: 'off'
    },

    // Start method is run when the Thing is constructed.
    start: function () {
        setInterval(()=> {
            testDevice.call('temp_data');
        }, 3000);

        // Turn on
        testDevice.call('turn_on');
    },

    turn_on: function () {
        testDevice.set('state', 'on');
    },

    turn_off: function () {
        testDevice.set('state', 'off');
    },

    temp_data: function () {
        let temp = Math.random() * 100;

        // Send data to the Grow-IoT app.
        testDevice.emit({
          type: 'temperature',
          value: temp
        });
    }
});

// Connects to localhost:3000 by default.
testDevice.connect();

```


# Working with hardware.

Grow.js works with most devices that can run node, and plays very well with the [Johnny-Five robotics library](http://johnny-five.io/), which has plugins for [a large number of devices](http://johnny-five.io/#platform-support). 

Note, with boards like the Tessel 2, Johnny-five is not required, but we're including it to make it easier to get started and support a wide variety of devices, sensors, and actuators.

### Wire up photo-resitor and led to arduino
Wire up your photo resistor and LED light like so:

![Wiring diagram](https://raw.githubusercontent.com/CommonGarden/Grow.js/master/examples/arduino/led-and-photoresistor/Arduino-night-light-circuit.png)

To use [Johnny-Five](http://johnny-five.io/), you need to make sure that your arduino is flashed with Standard Firmata. Instructions for doing so can be found [here](https://github.com/rwaldron/johnny-five/wiki/Getting-Started#trouble-shooting). Once that's done you're ready for the next step!

Take a look at the [led-and-photoresistor arduino example](https://github.com/CommonGarden/Grow.js/tree/master/examples/arduino/led-and-photoresistor) in the `examples/arduino/` folder.

Create a new thing in the Grow-IoT ui and copy and paste the UUID and Token into the example below.

<!-- THIS SHOULD BE AN EXAMPLE IN GROW-IOT as well... it should have a web component aspect. -->

```javascript
// Require the Grow.js build and johnny-five library.
var GrowInstance = require('Grow.js');
var five = require('johnny-five');

// Create a new board object
var board = new five.Board();

// When board emits a 'ready' event run this start function.
board.on('ready', function start() {
    // Define variables
    // Note: if you wire the device slightly differently you may need to
    // change the pin numbers below.
    var LED = new five.Pin(13);

    // Create a new grow instance.
    var grow = new GrowInstance({
        uuid: 'COPY_PASTE_UUID_HERE',
        token: 'COPY_PASTE_TOKEN_HERE',

        properties: {
            state: 'off'
        },

        turn_light_on: function () {
            LED.high();
            grow.set('state', 'on');
            console.log('light on');
        },

        turn_light_off: function () {
            LED.low();
            grow.set('state', 'off');
            console.log('light off');
        }
    });

    // Connects to localhost:3000 by default
    grow.connect();
});
```

Run the new `example.js` file with:

```bash
node examples/arduino/led-and-photoresistor/example.js
```

Note: on certain opperating systems you may need to prefix that command with `sudo` to allow the script access to USB.

# Connecting devices
### Host / Port
The host is where the device will be looking for a CommonGarden-IoT instance. By default the host is set to `localhost` and the port is set to Meteor's standard of `3000`. This will work nicely for usb devices like Arduino.

For connecting over wifi, connect your device to wifi and set the `host` to the IP address where the Grow-IoT instance is running. Pass the options to the `connect()` method like so:

```javascript
grow.connect({
    "host": "YOUR_IP_HERE"
})
```

#### Connecting over SSL
You can connect securely to our Grow-IoT alpha instance on https://grow.commongarden.org, or see the [Grow-IoT repo](https://github.com/CommonGarden/Grow-IoT) to easily start your own IoT network locally or hosted on [Meteor Galaxy](https://galaxy.meteor.com).

SSL is supported though will require a bit more setup. If you are hosting your instance off a computer with a dedicated IP address pass the following the `connect()` method.

```javascript
grow.connect({
    "host": "YOUR_IP_HERE",
    "port": 443,
    "ssl": true
})
```

If you are hosting on a cloud instance such as [Meteor Galaxy](https://galaxy.meteor.com), you might need specify the servername. The example below shows you how to connect securely to the instance at [grow.commongarden.org](https://grow.commongarden.org):

```javascript
grow.connect({
    "host": "grow.commongarden.org",
    "tlsOpts": {
        "tls": {
            "servername": "galaxy.meteor.com"
        }
    },
    "port": 443,
    "ssl": true
});
```

# Developing

Code is written in ES6, and compiled using [rollup](https://github.com/rollup/rollup).

`npm run build` builds the library.

`npm run test` builds the library, and runs tests in the test folder.

The documentation is written in jsdoc, built using [Mr-Doc](https://mr-doc.github.io/), and on the [gh-pages branch of this repo](https://github.com/CommonGarden/Grow.js/tree/gh-pages).

# Contributing

Please read:
* [Code of Conduct](https://github.com/CommonGarden/Organization/blob/master/code-of-conduct.md)
* [Contributing info](https://github.com/CommonGarden/Organization/blob/master/contributing.md)

## License
Grow.js is released under the 2-Clause BSD License, sometimes referred to as the "Simplified BSD License" or the "FreeBSD License".
